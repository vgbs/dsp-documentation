# Requesting a Login Page Banner

We can display an informational banner on the Keycloak login page (e.g., for planned maintenance or service
disruptions). To request one, please provide the following information.

## 1. Realm(s)

A realm is identified by **brand + country + B2B or B2C**. Tell us:

- **Brand** (e.g. Vaillant, Bulex, Saunier Duval, ...)
- **Country** (e.g. Netherlands, Germany, Belgium, ...)
- **B2B or B2C** (or both, if the banner should appear on more than one portal)

> Example: "Vaillant Netherlands B2B (myvaillantpro.nl)"

If you're not sure which realm your portal maps to, just give us the login page URL and we'll match it.

## 2. Message text

Provide the exact wording you want shown, **in the local language**. Optionally, you could also provide us an **English translation** so we can
verify meaning and formatting before publishing.

> Example (Dutch):
> We vernieuwen onze online systemen
> Om je nog sneller, efficiënter en beter van dienst te kunnen zijn, vernieuwen we onze online systemen.
> Daarom kun je van 27 augustus t/m 1 september niet inloggen op je account. Hierdoor zijn een aantal online
> diensten tijdelijk niet beschikbaar.
> Meer informatie vind je op onze speciale informatiepagina.
>
> English translation:
> We are renewing our online systems
> To serve you faster, more efficiently, and better, we are currently renewing our online systems.
> Because of this, you will not be able to log in to your account from August 27 through September 1. As a
> result, some online services will be temporarily unavailable.
> More information is available on our dedicated status page.

## 3. Maintenance window

- **Start date** of the maintenance/disruption
- **End date** of the maintenance/disruption

## 4. When the banner should go live

- **Publish date** - when the banner should start showing (this is often *before* the maintenance window itself,
  to give users notice)
- **Take-down date** - when the banner should be removed (often the same as the maintenance end date, but let us
  know if different)

## 5. Links

If the message should include a link (e.g., to a status page), provide:

- The **exact URL**
- The **exact word(s) or phrase** the link should be applied to (e.g. "informatiepagina")

## 6. Styling

Tell us simply whether any part of the text needs to be:

- **Bold**
- **Underlined**
- Both bold and underlined
- Or no styling at all - plain text

If styling is needed, please show us a styled message in the JIRA ticket.

We'll handle the technical implementation - you don't need to provide any HTML or code, only indicate which words
need which treatment.

If more styling options are needed, please let us know in advance, and we can discuss whether it can be accommodated.

---

## Summary checklist

Please confirm you've provided all the following:

- Brand
- Country
- B2B or B2C (or both)
- Message text (local language)
- (Optional) English translation of the message
- Maintenance window (start and end date)
- Banner publish date and take-down date
- Any links (URL + linked word/phrase)
- Styling needs (bold / underline / both / none), marked on the text