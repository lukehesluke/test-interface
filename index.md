# Open Booking API Test Interface

This is the specification and JSON-LD namespace for the test interface of the [Open Booking API](https://www.openactive.io/open-booking-api/EditorsDraft).

> Status: Draft | [Please Provide Feedback via GitHub](https://github.com/openactive/test-interface/issues)

## Status

This specification and namespace is in a draft state, however every effort will be made in subsequent revisions to maintain the IDs associated with the enumerations defined here.

## Overview

This interface must be implemented by the Booking System to make use of the "Controlled" test mode within the [OpenActive Test Suite](https://github.com/openactive/openactive-test-suite/). For the "Random" test mode, opportunities that match the criteria specified in the `TestOpportunityCriteriaEnumeration` enumeration must exist in the Booking System.

This interface and associated namespace is defined as a convenience to aid testing of the Open Booking API. Booking Systems MUST NOT expose this interface in production environments.

## Namespace

The namespace MUST be referenced using the URL `"https://openactive.io/test-interface"` (which will return the [JSON-LD definition](https://openactive.io/test-interface/test-interface.jsonld) if the `Accept` header contains `application/ld+json`), and is designed to be used in conjunction with the `"https://openactive.io/"` namespace.

## Test Interface Endpoints

The Test Interface Endpoints are specified relative to the same [Base URI](https://openactive.io/open-booking-api/EditorsDraft/#dfn-base-uri) that is defined in the Open Booking API.

### Datasets Endpoints

To use the "Controlled" test mode within the [OpenActive Test Suite](https://github.com/openactive/openactive-test-suite/), the `/test-interface/datasets` endpoints must be implemented. They are not required for "Random" test mode.

A `testDatasetIdentifier` is set in the configuration of the [OpenActive Test Suite](https://github.com/openactive/openactive-test-suite/) instance, and is reused across multiple test runs of the same instance. This allows any existing test data to be cleaned up, while still allowing multiple test suite instances to execute against a shared booking system environment simultaneously.

#### `DELETE /test-interface/datasets/:testDatasetIdentifier`

This endpoint deletes all opportunities within a given `testDatasetIdentifier`, and also deletes any `Order`s and `OrderItem`s associated with them.

The endpoint is called just once before each test run, when the [OpenActive Test Suite](https://github.com/openactive/openactive-test-suite/) is run in "Controlled" test mode.

It must clean up test data from previous test runs for the given `testDatasetIdentifier`.

##### Example Request

This request would delete all opportunities within the test dataset "`uat-ci`".

```javascript
DELETE /test-interface/datasets/uat-ci HTTP/1.1
Host: example.com
Date: Mon, 8 Oct 2018 20:52:35 GMT
Accept: application/vnd.openactive.booking+json; version=1
```

##### Example Response

```javascript
HTTP/1.1 204 No Content
Date: Mon, 8 Oct 2018 20:52:36 GMT
Content-Type: application/vnd.openactive.booking+json; version=1
```

#### `POST /test-interface/datasets/:testDatasetIdentifier/opportunities`

This endpoint creates an opportunity in the Booking System, that matches the specified criteria.

The endpoint is called (potentially multiple times) before each individual test starts executing, when the [OpenActive Test Suite](https://github.com/openactive/openactive-test-suite/) is run in "Controlled" test mode.

The endpoint must accept a [bookable opportunity type](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), which includes a specific `test:TestOpportunityCriteriaEnumeration` to which the newly created opportunity just conform, and the appropriate `@type` of `superEvent` or `facilityUse` to disambiguate the type of opportunity to be created. It must also include an `organizer` or `provider` `@id` to specify the Seller within which the opportunity should be created.

The Booking System must create an opportunity of the type specified in `@type` (taking into account `@type` of `superEvent` or `facilityUse`) matching the criteria specified by `test:TestOpportunityCriteriaEnumeration`, within the specified notional "Test Dataset" defined by the `testDatasetIdentifier` within the path.

##### Example Request

This request would create a new `ScheduledSession`, within the test dataset "`uat-ci`", that meets the criteria specified in `https://openactive.io/test-interface#TestOpportunityBookable`, for the `organizer` with `@id` `https://id.booking-system.example.com/organizer/3`.

```javascript
POST /test-interface/datasets/uat-ci/opportunities HTTP/1.1
Host: example.com
Date: Mon, 8 Oct 2018 20:52:35 GMT
Accept: application/vnd.openactive.booking+json; version=1

{
  "@context": [
    "https://openactive.io/",
    "https://openactive.io/test-interface"
  ],
  "@type": "ScheduledSession",
  "superEvent": {
    "@type": "SessionSeries",
    "organizer": {
      "@type": "Organization",
      "@id": "https://id.booking-system.example.com/organizer/3"
    }
  },
  "test:testOpportunityCriteria": "https://openactive.io/test-interface#TestOpportunityBookable"
}
```


##### Example Response

```javascript
HTTP/1.1 201 Created
Date: Mon, 8 Oct 2018 20:52:36 GMT
Content-Type: application/vnd.openactive.booking+json; version=1

{
  "@context": "https://openactive.io/",
  "@type": "ScheduledSession",
  "@id": "https://id.booking-system.example.com/scheduled-sessions/42"
}
```

### Actions Endpoint

#### `POST /test-interface/actions`

The `/test-interface/actions` endpoint is called to simulate a Booking System instigated action for the specified opportunity. 

The endpoint is called when the [OpenActive Test Suite](https://github.com/openactive/openactive-test-suite/) is run in both "Controlled" and "Random" test modes, only for those tests that depend on test action functionality being available in the Booking System.

If this endpoint is not implemented, the features whose tests depend on this endpoint must be configured to "disabled-tests" mode, to allow the test suite to run successfully.

The Booking System must respond with a `204` status code and an empty body to indicate success.

##### Example Request

This request would execute the simulation specified by `test:SellerRequestedCancellationSimulateAction` on the `ScheduledSession` with `@id` of `https://id.booking-system.example.com/session-series/42`.

```javascript
POST /test-interface/actions HTTP/1.1
Host: example.com
Date: Mon, 8 Oct 2018 20:52:35 GMT
Accept: application/vnd.openactive.booking+json; version=1

{
  "@context": [
    "https://openactive.io/",
    "https://openactive.io/test-interface"
  ],
  "@type": "test:SellerRequestedCancellationSimulateAction",
  "object": {
    "@type": "ScheduledSession",
    "@id": "https://id.booking-system.example.com/scheduled-sessions/42"
  }
}
```

##### Example Response

```javascript
HTTP/1.1 204 No Content
Date: Mon, 8 Oct 2018 20:52:36 GMT
Content-Type: application/vnd.openactive.booking+json; version=1
```


# Namespace



## Properties

| (Class) Property    |  Expected Type  | Description                                                         |
|---------------------|-----------------|---------------------------------------------------------------------|
| <a name="testOpportunityCriteria"></a> ([`schema:Event`](https://schema.org/Event)) <br/>  `test:testOpportunityCriteria` | [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | The opportunity criteria which the Event conforms to. |



## Classes

| Class                      | subClasses | Description                                                                                 |
|----------------------------|------------|---------------------------------------------------------------------------------------------|
| <a name="OpenBookingSimulateAction"></a> `test:OpenBookingSimulateAction` | [`schema:Action`](https://schema.org/Action) | An action that invokes a simulation of a specific behaviour within the booking system. |
| <a name="TestOpportunityCriteriaEnumeration"></a> `test:TestOpportunityCriteriaEnumeration` | [`schema:Enumeration`](https://schema.org/Enumeration) | An enumeration of test opportunity criteria to which an opportunity must conform. |
| <a name="AccessChannelUpdateSimulateAction"></a> `test:AccessChannelUpdateSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate an [`accessChannel` Update](https://www.openactive.io/open-booking-api/EditorsDraft/#other-notifications). |
| <a name="AccessCodeUpdateSimulateAction"></a> `test:AccessCodeUpdateSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate an [`accessCode` Update](https://www.openactive.io/open-booking-api/EditorsDraft/#other-notifications). |
| <a name="AccessPassUpdateSimulateAction"></a> `test:AccessPassUpdateSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate an [`accessPass` Update](https://www.openactive.io/open-booking-api/EditorsDraft/#other-notifications). |
| <a name="ChangeOfLogisticsLocationSimulateAction"></a> `test:ChangeOfLogisticsLocationSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate a location related [Change of Logistics](https://www.openactive.io/open-booking-api/EditorsDraft/#change-of-logistics-notifications). |
| <a name="ChangeOfLogisticsNameSimulateAction"></a> `test:ChangeOfLogisticsNameSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate a name related [Change of Logistics](https://www.openactive.io/open-booking-api/EditorsDraft/#change-of-logistics-notifications). |
| <a name="ChangeOfLogisticsTimeSimulateAction"></a> `test:ChangeOfLogisticsTimeSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate a time related [Change of Logistics](https://www.openactive.io/open-booking-api/EditorsDraft/#change-of-logistics-notifications). |
| <a name="CustomerNoticeSimulateAction"></a> `test:CustomerNoticeSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate a [Customer Notice](https://www.openactive.io/open-booking-api/EditorsDraft/#customer-notice-notifications). |
| <a name="OpportunityAttendanceUpdateSimulateAction"></a> `test:OpportunityAttendanceUpdateSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate an [Opportunity Attendance Update](https://www.openactive.io/open-booking-api/EditorsDraft/#opportunity-attendance-updates). |
| <a name="ReplacementSimulateAction"></a> `test:ReplacementSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate a [Replacement](https://www.openactive.io/open-booking-api/EditorsDraft/#cancellation-replacement-refund-calculation-and-notification). |
| <a name="SellerAcceptOrderProposalSimulateAction"></a> `test:SellerAcceptOrderProposalSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate [Proposal Approval of an `OrderProposal`](https://openactive.io/open-booking-api/EditorsDraft/#proposal-approval). |
| <a name="SellerRejectOrderProposalSimulateAction"></a> `test:SellerRejectOrderProposalSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate [Proposal Rejection of an `OrderProposal`](https://openactive.io/open-booking-api/EditorsDraft/#proposal-rejection). |
| <a name="SellerRequestedCancellationSimulateAction"></a> `test:SellerRequestedCancellationSimulateAction` | [`test:OpenBookingSimulateAction`](https://openactive.io/test-interface#OpenBookingSimulateAction) | Simulate a [Seller requested cancellation](https://www.openactive.io/open-booking-api/EditorsDraft/#seller-requested-cancellation). |
| <a name="SellerRequestedCancellationWithMessageSimulateAction"></a> `test:SellerRequestedCancellationWithMessageSimulateAction` | [`test:SellerRequestedCancellationSimulateAction`](https://openactive.io/test-interface#SellerRequestedCancellationSimulateAction) | Simulate a [Seller requested cancellation](https://www.openactive.io/open-booking-api/EditorsDraft/#seller-requested-cancellation), with a `cancellationMessage` provided. |



## Enumeration Values

| Type          | Value    | Description                                                                    |
|---------------|----------|--------------------------------------------------------------------------------|
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookable"></a> `https://openactive.io/test-interface#TestOpportunityBookable` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`. If both free and paid opportunities are supported by the Booking System, the resulting opportunity should be randomly either free or paid. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableAdditionalDetails"></a> `https://openactive.io/test-interface#TestOpportunityBookableAdditionalDetails` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and including at least one `Offer` with an `openBookingFlowRequirement` of `https://openactive.io/OpenBookingIntakeForm`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableAttendeeDetails"></a> `https://openactive.io/test-interface#TestOpportunityBookableAttendeeDetails` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and including at least one `Offer` with an `openBookingFlowRequirement` of `https://openactive.io/OpenBookingAttendeeDetails`. If both free and paid opportunities are supported by the Booking System, the resulting opportunity should be randomly either free or paid. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableCancellable"></a> `https://openactive.io/test-interface#TestOpportunityBookableCancellable` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and includes at least one `Offer` without `latestCancellationBeforeStartDate` and with `allowCustomerCancellationFullRefund` set to `true`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableCancellableOutsideWindow"></a> `https://openactive.io/test-interface#TestOpportunityBookableCancellableOutsideWindow` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and including at least one `Offer` with `allowCustomerCancellationFullRefund` set to `true`, except with `latestCancellationBeforeStartDate` outside of range. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableCancellableWithinWindow"></a> `https://openactive.io/test-interface#TestOpportunityBookableCancellableWithinWindow` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and including at least one `Offer` with `allowCustomerCancellationFullRefund` set to `true` and with `latestCancellationBeforeStartDate` in range. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableFiveSpaces"></a> `https://openactive.io/test-interface#TestOpportunityBookableFiveSpaces` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity == 5`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableFlowRequirementOnlyApproval"></a> `https://openactive.io/test-interface#TestOpportunityBookableFlowRequirementOnlyApproval` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, and including at least one `Offer` with a `openBookingFlowRequirement` value that contains only `https://openactive.io/OpenBookingApproval`. If both free and paid opportunities are supported by the Booking System, the resulting opportunity should be randomly either free or paid. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableFree"></a> `https://openactive.io/test-interface#TestOpportunityBookableFree` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, where at least one `Offer` has a `price` of `0`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableFreePrepaymentOptional"></a> `https://openactive.io/test-interface#TestOpportunityBookableFreePrepaymentOptional` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, where at least one `Offer` has a `prepayment` value of `https://openactive.io/Optional` and a `price` of  `0`. This criteria [violates the specification](https://openactive.io/open-booking-api/EditorsDraft/#free-opportunities), and is only used internally by the OpenActive Test Suite. An implementation of the Test Interface on the Booking System must return an error for this value. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableFreePrepaymentRequired"></a> `https://openactive.io/test-interface#TestOpportunityBookableFreePrepaymentRequired` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, where at least one `Offer` has a `prepayment` value of `https://openactive.io/Required` and a `price` of  `0`. This criteria [violates the specification](https://openactive.io/open-booking-api/EditorsDraft/#free-opportunities), and is only used internally by the OpenActive Test Suite. An implementation of the Test Interface on the Booking System must return an error for this value. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableNoSpaces"></a> `https://openactive.io/test-interface#TestOpportunityBookableNoSpaces` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity == 0`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableNonFree"></a> `https://openactive.io/test-interface#TestOpportunityBookableNonFree` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and includes at least one `Offer` with `Offer.price > 0`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableNonFreePrepaymentOptional"></a> `https://openactive.io/test-interface#TestOpportunityBookableNonFreePrepaymentOptional` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, where at least one `Offer` has a `prepayment` value of `https://openactive.io/Optional` and a `price` greater than  `0`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableNonFreePrepaymentRequired"></a> `https://openactive.io/test-interface#TestOpportunityBookableNonFreePrepaymentRequired` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, where at least one `Offer` has either no `prepayment` value specified or a `prepayment` value of `https://openactive.io/Required`, and a `price` greater than  `0`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableNonFreePrepaymentUnavailable"></a> `https://openactive.io/test-interface#TestOpportunityBookableNonFreePrepaymentUnavailable` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, where at least one `Offer` has a `prepayment` value of `https://openactive.io/Unavailable` and a `price` greater than  `0`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableNonFreeTaxGross"></a> `https://openactive.io/test-interface#TestOpportunityBookableNonFreeTaxGross` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, and including at least one `Offer` with `Offer.price > 0`, and where `taxMode` of the `seller` is `https://openactive.io/TaxGross`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableNonFreeTaxNet"></a> `https://openactive.io/test-interface#TestOpportunityBookableNonFreeTaxNet` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, and including at least one `Offer` with `Offer.price > 0`, and where `taxMode` of the `seller` is `https://openactive.io/TaxNet`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableNotCancellable"></a> `https://openactive.io/test-interface#TestOpportunityBookableNotCancellable` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and includes at least one `Offer` with `latestCancellationBeforeStartDate` outside of range, or with `allowCustomerCancellationFullRefund` set to `false`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableOutsideValidFromBeforeStartDate"></a> `https://openactive.io/test-interface#TestOpportunityBookableOutsideValidFromBeforeStartDate` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and a `startDate` outside of range of `validFromBeforeStartDate`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableOutsideValidFromBeforeStartDateWindow"></a> `https://openactive.io/test-interface#TestOpportunityBookableOutsideValidFromBeforeStartDateWindow` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and including at least one `Offer` with `validFromBeforeStartDate` provided, except where `validFromBeforeStartDate` subtracted from the `startDate` is in the future. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableSellerTermsOfService"></a> `https://openactive.io/test-interface#TestOpportunityBookableSellerTermsOfService` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `organizer` or `provider` containing a `termsOfService` property. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableUsingPayment"></a> `https://openactive.io/test-interface#TestOpportunityBookableUsingPayment` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and includes at least one `Offer` with `Offer.price > 0` and a `prepayment` value that is not `https://openactive.io/Unavailable`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableWithinValidFromBeforeStartDate"></a> `https://openactive.io/test-interface#TestOpportunityBookableWithinValidFromBeforeStartDate` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and a `startDate` in range of `validFromBeforeStartDate`. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityBookableWithinValidFromBeforeStartDateWindow"></a> `https://openactive.io/test-interface#TestOpportunityBookableWithinValidFromBeforeStartDateWindow` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1` and including at least one `Offer` with `validFromBeforeStartDate` provided, and where `validFromBeforeStartDate` subtracted from the `startDate` is in the past. |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityNotBookableViaAvailableChannel"></a> `https://openactive.io/test-interface#TestOpportunityNotBookableViaAvailableChannel` | availableChannel of the `Offer` does not include `https://openactive.io/OpenBookingPrepayment` |
| [`test:TestOpportunityCriteriaEnumeration`](https://openactive.io/test-interface#TestOpportunityCriteriaEnumeration) | <a name="TestOpportunityOnlineBookable"></a> `https://openactive.io/test-interface#TestOpportunityOnlineBookable` | [Bookable Opportunity with Bookable Offer](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), with `remainingAttendeeCapacity > 1`, including at least one `Offer` with a `openBookingFlowRequirement` value that contains only `https://openactive.io/OpenBookingApproval`, and has `eventAttendanceMode` equal to `MixedEventAttendanceMode` or `OnlineEventAttendanceMode`. If both free and paid opportunities are supported by the Booking System, the resulting opportunity should be randomly either free or paid. |
