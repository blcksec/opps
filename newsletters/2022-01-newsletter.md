# Technical newsletter for developers 2022-01

💥 DRAFT! 💥

This newsletter was sent in January 2022.

* [Recurring API: The "PROCESSING" status](#recurring-api-the-processing-status)
* [Please check your eCom API calls](#please-check-your-ecom-api-calls)
* [Vipps Login directly from phone number and QR code](#vipps-login-directly-from-phone-number-and-QR-code)
* [Deprecation of the Vipps Signup API](#deprecation-of-the-vipps-signup-api)
* [Reminders](#reminders)
  * [Use the API Dashboard to find problems with your integration](#use-the-api-dashboard-to-find-problems-with-your-integration)
  * [Use Statuspage to get information about incidents](#use-statuspage-to-get-information-about-incidents)
  * [Omikron tips](#omikron-tips)
  * [Vipps Hurtigkasse: Use the explicit flow](#vipps-hurtigkasse-use-the-explicit-flow)
  * [Use Userinfo to register visitors](#use-userinfo-to-register-visitors)
  * ["Click and collect" recommendations](#click-and-collect-recommendations)
* [Newsletter archive](#newsletter-archive)
* [Questions or comments?](#questions-or-comments)

# Recurring API: The "PROCESSING" status

We would like to emphasize: With the Vipps Recurring API merchants ask Vipps
to make the charges, and Vipps handles _everything_ for the merchant.

Vipps does not "leak" the customers's information about insufficient funds,
blocked cards, etc. Users are informed about all such problems in Vipps, which
is the only place they can be corrected. The merchant's customer service should
always ask the user to check in Vipps if a charge has failed.

We have, for a while, attempted to give the merchant up-to-date information
about the status of the charge, with `failureReason` and `failureDescription`,
but this causes more confusion than clarity: Even if one charge attempt fails,
the cvarge itself has not failed until all the attempts are completed.
We will therefore continue to use `PROCESSING`, as we have done, but until
all charge attempts have been made.

The status of a charge will be `PROCESSING` while Vipps is taking care of business,
from the `due` date until the `retryDays` have passed. After that the status
will be `CHARGED` or `FAILED`. See the API documentation for more details.

See:
[Charge states](https://github.com/vippsas/vipps-recurring-api/blob/processing/vipps-recurring-api.md#charge-states).

# Please check your eCom API calls

We see that a lot of calls to
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST)
use an incorrectly formatted phone number.
The effect is that the user's phone number is not correctly pre-filled on
the Vipps landing page.

Please make sure you send the `mobileNumber` in `91234567` format, not
`+47 91 23 45 67` or something else.

We have previously tried to respond with `HTTP 400 Bad Request` (as we should)
for incorrectly formatted phone numbers, but that broke _a lot_  of integrations,
so we decided to accept the incorrect API calls even though they give a poor
user experience.

# Vipps Login directly from phone number and QR code

The Vipps Login API now supports merchant-initiated login directly from a phone
number, and by users scanning a QR code.

* Vipps Login directly from phone number is a Client Initiated Backchannel
Authentication (CIBA) flow where authentication/registration does not start in a
browser or an app, but is initiated by the merchant. The user simply confirms in Vipps.
This is typically in physical contexts like point of sales (POS), on the phone
in call center solutions, or using devices/terminals like TV boxes. The user can
either get a confirmation in Vipps, or be taken to the merchant's confirmation page in a browser.

* Vipps Login directly from QR code allows the user to scan a QR code with the
phone's camera or with Vipps and register or log in directly. The user can either get a confirmation in Vipps,
or be taken to the merchant's confirmation page in a browser.
This can be used in marketing, from posters, screens etc.

More information on both possibilities can be found in the
[Vipps Login API](https://github.com/vippsas/vipps-login-api)
documentation.

# Deprecation of the Vipps Signup API

The old API that some partners still use to sign up new merchants will
be phased out. See
[Deprecation of the Vipps Signup API](https://github.com/vippsas/vipps-signup-api/blob/master/vipps-signup-api-deprecation.md)
and
[Vipps Partners](https://github.com/vippsas/vipps-partner).

# Reminders

## Use the API Dashboard to find problems with your integration

The API Dashboard is available for both the production and test environments,
and is an easy way to see if you are using the Vipps APIs correctly.
Think of it as a "health check", that you can use to see if there are any
problems you need to investigate.

See it on
[portal.vipps.no](https://portal.vipps.no)
under the "Utvikler" ("Developer") tab.
Here's an example for the Vipps eCom API's `/refund` endpoint:

![API Dashboard example](images/2021-02-api-dashboard-example.png)

## Use Statuspage to get information about incidents

Vipps uses Statuspage to inform about problems, planned maintenance, etc.
You can subscribe to get all updates, and also subscribe to specific incidents.

One example:
[Apache Log4j vulnerability – No impact to Vipps](https://vipps.statuspage.io/incidents/yfbhp4lm9g4j).

See:
[Statuspage for the production environment](https://vipps.statuspage.io)
and
[the other status pages](https://github.com/vippsas/vipps-developers#status-pages).

## Omikron tips

These Vipps solutions are extra relevant (again):

- Use
  [Vipps Logg inn](https://vipps.no/produkter-og-tjenester/privat/logg-inn-med-vipps/logg-inn-med-vipps/)
  and the
  [Vipps Login API](https://github.com/vippsas/vipps-login-api)
  to register visitors - it's free.
- [Use Userinfo to register visitors](#use-userinfo-to-register-visitors)
  as an easy-to-use step in a normal Vipps payment.
- ["Click and collect" recommendations](#-click-and-collect--recommendations)
  to speed up the user experience for your customers.

## Vipps Hurtigkasse: Use the explicit flow

When users are prompted to select shipping address and shipping address, the
explicit flow is _strongly_ recommended. The user then has to actively
select shipping address and shipping method.

The "old" flow does not prompt the user in the same way, and some users
do not notice that they select an incorrect/old/outdated address.

Using the explicit flow is simple: Just specify
`"useExplicitCheckoutFlow": true`
in
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST).

See
[Old and new express checkout flow](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#old-and-new-express-checkout-flow)
for more details.

## Use Userinfo to register visitors

For guest registration/tracking: Use _Userinfo_ to ask for the user's details, such as:
phone number, name, email address, postal address, birth date, national identity number and bank accounts.
The user must of course consent to sharing the information.

See
[Userinfo for eCom](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#userinfo)
and
[Userinfo for Recurring](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md#userinfo).

## "Click and collect" recommendations

For "click and collect" we recommend to use
[`staticShippingDetails`](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#shipping-and-static-shipping-details)
(to avoid the extra HTTP roundtrip where Vipps asks the merchant
for the shipping options and prices) and also to set the default
shipping method to "Click and collect".

This will significantly speed up the payment process for customers.

This is done in the
[`POST:​/ecomm​/v2​/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)
call by including:

```
"staticShippingDetails": [
  {
    "isDefault": "Y",
    "priority": 0,
    "shippingCost": 0,
    "shippingMethod": "Click and collect",
    "shippingMethodId": "click-and-collect"
  }
]
```


# Newsletter archive

All the previous newsletters are in the
[newsletter archive](https://github.com/vippsas/vipps-developers/tree/master/newsletters).

# Questions or comments?

We're always happy to help with code or other questions you might have!
Please create [GitHub issues or pull requests](https://github.com/vippsas)
for the relevant API,
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).