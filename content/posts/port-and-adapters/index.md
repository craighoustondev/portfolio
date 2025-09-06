---
title: "Separate external dependencies from your domain with Ports & Adapters"
date: 2025-09-06
draft: true
summary: "How I used the Ports & Adapters architecture to keep the domain free of the internal details of third-party data providers."
---

One of the pitfalls when introducing a dependency on an external system is that details leak from the dependency in to your domain.

I am going to tell you about how I used the Ports & Adapters architecture to keep the domain free of the internal details of third-party data providers.

## What is meant by domain?
> The subject area to which the user applies a program is the domain of the software - <a href="https://www.domainlanguage.com/wp-content/uploads/2016/05/DDD_Reference_2015-03.pdf" target="_blank" rel="noopener noreferrer">Eric Evans, 2015</a>
> 

The domain sits at the core of a software system and defines and encodes the business rules you care about. It’s the code you have full control over. The logic you invested time in understanding and refining and (hopefully) made your customers lives better. An algorithm that helps schedule your users’ daily activities is part of your domain. The data provider you started pulling data from is not part of your domain as you have no control over how their API works or the data they send you.

## Why it’s important to separate external dependencies from the domain

In my experience, infiltration of the details of an external dependency into the domain has caused a number of problems.

### Third-party is reflected in the implementation

In the specific example I want to explore, we were pulling company data from a single third-party and representing that data in our UI. The entire raw data object received from the third-party was passed in to the Django template that generated the HTML for the UI. The template was overwhelmed with code that referenced complex paths within the data that were determined by the provider. The implementation was bending to accommodate the third-party data, rather than the other way around. If the shape of the data was ever to change, there was a high chance the code would error and there would be no way of knowing until a user accessed the page.

```html
<section class="business-section report-generated">
    <h2>Company summary</h2>
    <article>
        <dl>
            <dt>Report Generated</dt>
            <dd>{{object.created_at|date:"d/m/Y"}}</dd>
            <dt>Status</dt>
            <dd>{{report.companySummary.companyStatus.status}}</dd>
            <dt>Status Description</dt>
            <dd>{{report.companySummary.companyStatus.description}}</dd>
            <dt>Employee Count</dt>
            <dd>{{report.otherInformation.employeesInformation.0.numberOfEmployees|default:"-"}}</dd>
            <dt>Company Number</dt>
            <dd>{{report.companySummary.companyRegistrationNumber}}</dd>
            <dt>VAT Number</dt>
            <dd>{{report.companyIdentification.basicInformation.vatRegistrationNumber}}</dd>
            <dt>Registration Date</dt>
            <dd>{{report.companyIdentification.basicInformation.companyRegistrationDate|parse_date|date:"d/m/Y"}}</dd>
            <dt>Type</dt>
            <dd>{{report.companyIdentification.basicInformation.legalForm.description}}</dd>
            <dt>Account Filing Date</dt>
            <dd>{{report.additionalInformation.miscellaneous.filingDateOfAccounts|parse_date|date:"d/m/Y"}}</dd>
            <dt>Registered Address</dt>
            <dd>{{report.contactInformation.mainAddress.simpleValue}}</dd>
        </dl>
    </article>
</section>

```

In addition to this, the name of the provider was littered throughout the codebase. Models, views, templates, forms and serializers all referenced the name of the provider. Even the publicly facing urls contained the provider name! We were taking the easy option of using their name rather than focussing on more meaningful domain language.

### Testing

In our code, the retrieval and storing of the data was tightly-coupled to the third-party. Any time we wanted to test basic internal logic around how and when we initiated the request we had to unnecessarily mock the external API calls. Mocking methods were inconsistent, adding to the complexity of what should have been very basic tests.

```python
class ReportTests:
    def test_generates_report_for_company(client, httpx_mock):
        httpx_mock.add_response(
		        url="https://provider_one.com/authenticate",
		        json={"token": "dummy_token"},
		    )
		    httpx_mock.add_response(
		        url="https://provider_one.com/companies/123123",
		        json={"name": "Acme Ltd"},
		    )

        response = client.post("/provider_one_report/", {
            "country_iso_code": "GB",
            "company_id": "123",
        })

        assert response.json()["name"] == "Acme Ltd"
```

### Difficult to implement a new provider

In early 2024 we needed to implement a new data provider in order to improve the quality of the data we presented back to our users. We had to dynamically select which provider to use based on some pre-defined conditions. It was almost impossible to re-use the existing code without increasing its complexity by scattering conditional logic throughout. We needed to find a solution that could set a standard and be re-used for other parts of our product that faced similar issues.

Enter Ports & Adapters.

## Ports & Adapters

<div style="display:flex; justify-content:center;">
    {{< figure src="ports-and-adapters.jpg">}}
</div>

The Ports & Adapters architecture (originally named Hexagonal architecture) was introduced by Alistair Cockburn in a 2005 <a href="https://alistair.cockburn.us/hexagonal-architecture" target="_blank" rel="noopener noreferrer">article</a>. Cockburn describes an architecture where the application (or domain) sits in the centre with a strict boundary between it and any outside concerns. The application can send or receive signals via ports to an adapter which translates the signal necessary for/from the outside technology. He cites GUIs, databases and third-party services as examples of outside concerns.

In the same article, Cockburn states the primary intent of the architecture:

> Allow an application to equally be driven by users, programs, automated test or batch scripts, and to be developed and tested in isolation from its eventual run-time devices and databases.
> 

## How we implemented Ports & Adapters

### Create a port

The first step for us was to define the port which would act as a contract for how our application would communicate with the company data providers. We implemented this as an abstract class in Python, analogous to an interface.

```python
from abc import ABC, abstractmethod

class CompanyReportGenerator(ABC):
    @abstractmethod
    def generate_report(
        self,
        country_iso_code: str,
        company_id: str,
    ) -> dict:
        pass
```

Python ABCs (Abstract Base Classes) don’t exactly equate to interfaces in strongly-typed languages such as Java or C# and some would argue that Python’s duck-typing makes this class unnecessary. In our case, I valued the fact that creating this forced us to think at an early stage about how our domain would communicate with the outside world and considered the overhead to be worth it.

### Define the adapter for the existing provider

Next we created an adapter that respected this contract and defined the provider-specific implementation to retrieve the necessary data.

```python
class ProviderOneCompanyReportGenerator(CompanyReportGenerator):
    def generate_report(
        self,
        country_iso_code: str,
        company_id: str,
    ) -> dict:
        try:
            data = ProviderOneAPI().credit_report(
                company_id,
                country_iso_code,
            )
        except ProviderOneRequestError:
		    _logger.info(f"Error fetching data from ProviderOne")
            raise
        return data
```

```python
class ProviderOneAPI:
    # code omitted for brevity
        
    @async_to_sync
    async def credit_report(self, country_iso_code: str, company_id: str) -> dict:
        if country_iso_code == "DE":
            return await self.get(
                f"/companies/{company_id}?country_specific_args"
            )
        elif country_iso_code == "BR":
            return await self.get(f"/companies/{company_id}?country_specific_args")

        return await self.get(f"/companies/{company_id}")
        
    async def get(self, path: str, **kwargs) -> dict:
        url = self.make_url(path)
        response = await self.http.get(
            url,
            params=kwargs,
            headers={
                "Authorization": f"Bearer {self.token}",
                "Accepts": "application/json",
            },
        )
        if 200 <= response.status_code < 300:
            return response.json()
        else:
            raise ProviderOneRequestError(response)
```

This gave us a clean, reusable interface for generating a company report with the existing provider. Now we needed a suitable place to call this from.

### Application service

We wanted to create a reusable single-entry point for generating a company report that did not need to concern itself with the internal details of how we are communicating with the outside world.

We utilised dependency injection to set the specific adapter to use. We knew that this would be beneficial when we came to implement other providers as the service code would not need to change. All we would need to do is dynamically decide which provider to use from the code that called the service.

```python
class ReportGenerator:
    def __init__(
        self,
        provider: CompanyReportGenerator, # Inject the relevant adapter
        country_iso_code: str,
        company_id: str,
    ):
        self.provider = provider
        self.country_iso_code = country_iso_code
        self.company_id = company_id

    def generate_report(self) -> dict:
        return self.provider.generate_report(
            self.country_iso_code,
            self.company_id,
        )
```

### Call the service

The action to initiate the generation of a company report was via an API request sent from our UI. We updated our Django view to select the adapter to use and then call the new service. This was straightforward at this point because we only had an adapter for one provider.

```python
class CompanyReportViewSet:
    @action(detail=False, methods=["post"])
    def generate_company_report(self, request):
        provider = ProviderOneCompanyReportGenerator() # Determine the provider
        report = ReportGenerator(
            provider=provider,
            country_iso_code=request.data.get("country_iso_code"),
            company_id=request.data.get("company_id"),
        ).generate_report()

        return Response(status=200, data=report)
```

We now had parity with the existing behaviour and were able to ship these changes without our customers knowing that the implementation had changed.

We had set ourselves up nicely for the next task of implementing the new provider.

### Repeat for new provider

We needed to create an adapter for the new provider which respected the same contract as the original adapter.

```python
class ProviderTwoCompanyReportGenerator(CompanyReportGenerator):
    def generate_report(
        self,
        country_iso_code: str,
        company_id: str,
    ) -> dict:
        report_data = ProviderTwoAPI().company_report(company_id)
        other_data = ProviderTwoAPI().other_data(company_id)
        
        # code omitted for brevity
```

```python
class ProviderTwoAPI:
    # code omitted for brevity
        
    @async_to_sync
    async def company_report(self, company_id: str):
        return await self.get(
            f"/data/{company_id}?other_necessary_args"
        )
        
    @async_to_sync
    async def other_data(self, company_id: str | None = None) -> dict:
        return await self.get(
            f"/other_data/{company_id}?other_necessary_args"
        )
        
    async def get(self, path):
        url = self.make_url(path)
        response = await self.http.get(
            url,
            params=None,
            headers={
                "Authorization": f"Bearer {self.token}",
                "Accepts": "application/json",
            },
        )
        if response.status_code == 204:
            return {}

        return response.json()
```

Notice that the implementation for generating a company report is different for the new provider as we need to make 2 API calls. The great thing about our new architecture was that we could easily implement this in isolation from the original provider and the service does not need to know that the implementation differs between the two.

Now we could implement the real logic for deciding which provider to use with minimal code change. The logic we wanted to implement was that reports for companies in the United Kingdom would use the original provider and other countries would use the new provider.

```python
class CompanyReportViewSet:
    @action(detail=False, methods=["post"])
    def generate_company_report(self, request):
        country_iso_code = request.data.get("country_iso_code")
        provider = ProviderOneCompanyReportGenerator()  # UK uses original provider
        if country_iso_code != "GB":
            provider = ProviderTwoCompanyReportGenerator()  # Non-UK uses new provider
        report = ReportGenerator(
            provider=provider,
            country_iso_code=country_iso_code,
            company_id=request.data.get("company_id"),
        ).generate_report()

        return Response(status=200, data=report)
```

We initially added this logic in the view for simplicity but we came to realise that our views should be as thin as possible and not contain business logic.

## Did this solve our problems?

In short, yes. We had pushed the provider specific implementation out of our domain and introduced naming that reflected the domain concepts and operations.

We were now able to test our internal logic free from the overhead of having to mock calls to the external services.

Once we had the architecture in place we were able to add a new provider with little code changes in our domain logic. We knew that we had future-proofed ourselves if the requirement arose to add yet another provider.

## Closing thoughts

There are a multitude of ways we could have tackled this, each with their own pros and cons. I like to apply pragmatism when making these choices rather than advocating for a one size fits all solution. We chose Ports & Adapters as it promised to solve the particular issues we were facing and is a well established architectural pattern with lots of examples to follow.

Using this pattern served us well at the time and has gone on to be replicated in other parts of our codebase with equal success.