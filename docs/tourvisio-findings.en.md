# TourVisio documentation findings

> Türkçe: [tourvisio-findings.md](tourvisio-findings.md)

The facts this plan relies on. The README tags claims as **[DOC]** (stated in the docs), **[SAMPLE]** (seen only in a sample JSON, not treated as a contract), or **[Q-n]** (not in the docs, see [open questions](../README.en.md#questions-for-tourvisio)).

## Facts from the documentation

| # | Finding | Source |
|---|---|---|
| D1 | **Backend → TourVisio only** login: `POST /api/authenticationservice/login`. Input `Agency`, `User`, `Password`. Output `token`, `expiresOn`, `tokenId`, `userInfo`. Next.js → backend auth is our own design | [Authentication](https://docs.santsg.com/tourvisio/authentication/) |
| D2 | The token is sent with `Authorization: Bearer`. Token lifetime is not stated. **TourVisio has no refresh token**, so a session can only be extended by calling `login` again | [Authentication](https://docs.santsg.com/tourvisio/authentication/) |
| D3 | `POST /api/productservice/getdepartureautocomplete` and `getarrivalautocomplete`. The arrival call takes `DepartureLocations` | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D4 | The autocomplete `type` and the search `type` are different enums. Airport maps 3 → 5, city maps 1 → 2 | [Enumerations](https://docs.santsg.com/tourvisio/enumarations/) |
| D5 | `POST /api/productservice/pricesearch`. The alternative `flightpricesearch` is labeled "Optional - To improve performance" | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D6 | `ServiceTypes`: `"1"` one-way, `"2"` round-trip. Round-trip uses `CheckIn` + `Night`. The recommended sample sends `supportedFlightReponseListTypes: [3]` (spelled as in the docs) | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D7 | The response contains `body.searchId`, `body.expiresOn`, and `body.flights[]`. **`offers[]` is inside each `flight` object** and has its own `expiresOn` field | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D8 | For round trips, outbound and return offers are paired by the intersection of their `groupKeys`, and the correct `offerId` is selected from `offerIds[]` | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D9 | A total price rule is defined: the higher price for packaged offers, the sum for non-packaged offers, then the `fees` rule | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D10 | `POST /api/productservice/getoffers` is optional and takes `searchId` + `offerIds` | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D11 | `POST /api/flightservice/getfarerules` requires `transactionId` or `reservationNumber`, plus `serviceId` + `segment.id`. It is defined for "third party flight (SanFlight)" services | [Flight Service](https://docs.santsg.com/tourvisio/flightservice/) |
| D12 | Response envelope: `header { requestId, success, messages[] }` + `body`. Message types: 1 Error, 2 Success, 3 Information, 4 Warning | [General](https://docs.santsg.com/tourvisio/general/) |

## Gaps and contradictions

- **Contradiction:** the Authentication page says the token must be sent on every request, but the `pricesearch` table says "Token Required: NO". This plan sends the token on every call (Q-2).
- **Undocumented:** rate limits, quotas, timeouts, retries, an error code list, and server-side pagination. `enablePaging: false` appears in responses without explanation.
- **[SAMPLE] Lifetime:** in two search samples, `expiresOn` is about 210 minutes after `responseTime`. We do not rely on this value and read it from each response.
- **[SAMPLE] Offer lifetime:** in both full search samples, every offer's `expiresOn` equals `body.expiresOn`. The docs do not say whether they can differ, so the plan handles both cases (Q-4).
- **[SAMPLE] Price:** sample prices, passenger counts, and currencies do not line up with each other. We treat the samples as examples of which fields may be present, with illustrative values (Q-11).
- **[SAMPLE] Dates:** some sample dates have no offset, while others end in `Z`. The time filter does no time zone conversion.
- **[SAMPLE] Stops:** one sample shows a single-segment, 75-minute flight with `stopCount: 1`. The meaning of the stop count must be confirmed (Q-12).
- **[SAMPLE] Non-stop flag:** search samples send `showOnlyNonStopFlight: false`, but the docs do not describe the field or its default (Q-16).
