# ROSHI API documentation

ROSHI has 2 ways of communicating with other clients:

- Webhooks
- REST API

## Webhooks

Webhooks allows ROSHI to send notifications to other clients.
You can configure a webhook URL at: https://app.roshi.sg/lender/settings in the notification tab.

Here are the different type of webhooks that ROSHI sends:

- NEW_LOAN_REQUEST: When a new lead is available
- APPOINTMENT_REQUESTED: When a borrower sets an appointment
- APPOINTMENT_CANCELED: When a borrower cancels an appointment
- OFFER_SELECTED: When a borrower selects an offer

## Webhook payload examples

### NEW_LOAN_REQUEST

```json
{
  "type": "NEW_LOAN_REQUEST",
  "data": {
    "loanRequestId": "4df3233a-98f8-49c5-8c01-5949359d8a13",
    "loanRequest": {
      "id": "4df3233a-98f8-49c5-8c01-5949359d8a13",
      "status": "ACTIVE",
      "purpose": "CREDIT_CARD_DEBT",
      "amount": 25000,
      "term": 12,
      "loanResponses": null,
      "createdAt": "2024-11-29T03:50:03.186Z",
      "approvedAt": null,
      "publicNote": null,
      "applicantInfo": {
        "age": 32,
        "bankDebt": 1000,
        "lenderDebt": 0,
        "monthlyIncome": 1000,
        "employmentStatus": "STUDENT",
        "jobTitle": "",
        "nric": "S0290695C",
        "currentEmploymentTime": "NA",
        "previousEmploymentTime": "NA",
        "propertyOwnership": "HDB",
        "residencyStatus": "PERMANANT_RESIDENT",
        "fullname": null,
        "phoneNumber": null,
        "postalCode": null,
        "id": "d110c7a6-3398-4e8b-93f3-bdd01bbbb6b1",
        "documents": [],
        "mlcbRatio": 0,
        "singpassData": {
          "cpfcontributions": {
            "source": "1",
            "history": [
              {
                "date": {
                  "value": "2022-10-08"
                },
                "month": {
                  "value": "2022-10"
                },
                "amount": {
                  "value": 2035
                },
                "employer": {
                  "value": "Employer 1"
                }
              }
            ],
            "lastupdated": "2024-01-26",
            "classification": "C"
          }
        }
      },
      "guarantorInfo": null
    }
  }
}
```

### OFFER_SELECTED

```json
{
  "type": "OFFER_SELECTED",
  "data": {
    "loanResponseId": "013c0c6f-f9c9-4fa1-915c-1adb5506ff5c",
    "loanRequestId": "4df3233a-98f8-49c5-8c01-5949359d8a13",
    "userId": "fd06e0b6-832c-4fb1-83e7-d052772781d2",
    "user": {
      "id": "fd06e0b6-832c-4fb1-83e7-d052772781d2",
      "name": "BERNARD LI GUO HAO",
      "phone": "65999999"
    }
  }
}
```

### APPOINTMENT_SCHEDULED

```json
{
  "type": "APPOINTMENT_SCHEDULED",
  "data": {
    "appointmentId": "2a69f5e0-4c26-4422-a2f1-bf95c5b1561e",
    "loanResponseId": "013c0c6f-f9c9-4fa1-915c-1adb5506ff5c",
    "loanRequestId": "4df3233a-98f8-49c5-8c01-5949359d8a13",
    "appointment": {
      "scheduledTime": "2024-11-30 12:30", //SGT TIME
      "store": {
        "name": "store name",
        "address": "store address",
        "id": "aaa45bca-e988-4c9e-83ca-2386f633dc20"
      }
    }
  }
}
```

### APPOINTMENT_CANCELED

```json
{
  "type": "APPOINTMENT_CANCELED",
  "data": {
    "appointmentId": "fc8c43f1-bfe6-4ad5-94cc-1ab1034d6ca5",
    "loanResponseId": "013c0c6f-f9c9-4fa1-915c-1adb5506ff5c",
    "loanRequestId": "4df3233a-98f8-49c5-8c01-5949359d8a13"
  }
}
```

## REST API

The API allows third party clients to execute actions on the ROSHI platform such as approving or rejecting an offer, or fetching data.

In order to use the API, you must be authenticated using an API Token.
An API token can be created at https://app.roshi.sg/lender/settings in the API tab.
The token must be passed with each API call in the header with the format:

```json
{
  "Authorization": "Bearer ${token}"
}
```

The base url for the ROSHI api is https://roshi-api.azurewebsites.net/api/v1/

### Endpoints:

#### Accept/Reject an offer:

- Method: POST
- URL: https://roshi-api.azurewebsites.net/api/v1/loan-offer/
- Request body:

```ts
const createLoanOfferBody = z.object({
  loanRequestId: z.string(),
  comment: z.string().optional(), //Optional comment
  status: z.enum(["ACCEPTED", "REJECTED"]), //Send string ACCEPTED or REJECTED
  offer: z
    .object({
      amount: z.number().int().positive(), //flat value (100 for 100$)
      term: z.number().int().positive(), //months (1 for 1 month)
      monthlyInterestRate: z.number().nonnegative(), //percentage (3 for 3%)
      fixedUpfrontFees: z.number().nonnegative(), //Flat value (100 for 100$)
      variableUpfrontFees: z.number().nonnegative(), //percentage (10 for 10%)
    })
    .optional(), //Offer only required when status is “ACCEPTED”
});
```

#### Delete an offer previously made:

- Method: DELETE
- URL: https://roshi-api.azurewebsites.net/api/v1/loan-offer/{ID}

#### Close a lead (Deal closed or rejected)

- Method: POST
- URL: https://roshi-api.azurewebsites.net/api/v1/loan-offer/close
- Request body:

```ts
export const closeLoanOfferSchema = z.object({
  loanRequestId: z.string(),
  outcomeStatus: z.enum(["APPROVED", "REJECTED"]),
  outcomeComment: z.string().optional(), //Required when outcomeStatus is “REJECTED”
  disbursementDate: z.string().datetime(),
  closedDealOffer: z
    .object({
      amount: z.number().int().positive(), //flat value (100 for 100$)
      term: z.number().int().positive(), //months (1 for 1 month)
      monthlyInterestRate: z.number().nonnegative(), //Percentage (3 for 3%)
      fixedUpfrontFees: z.number().nonnegative(), //Flat value (100 for 100$)
      variableUpfrontFees: z.number().nonnegative(), //Percentage (10 for 10%)
    })
    .optional(), //Offer only required when outcomeStatus is “APPROVED”,
});
```
