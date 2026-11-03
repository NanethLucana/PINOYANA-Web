# PINOYANA EmailJS + Gmail setup

The website's order and feedback forms use EmailJS's browser SDK. The Gmail password is never used in the website. EmailJS's public key is intended for browser use; restrict the EmailJS account to the website origin and protect the account with two-factor authentication.

## Confirm Gmail service and get the account key

1. Sign in to [EmailJS](https://www.emailjs.com/) and confirm the service with ID `service_0fri38o` is the intended Gmail service under **Email Services**. Do not create another service or enter a Gmail password into the website.
2. The website uses the exact Service ID `service_0fri38o`, not the `primary_service` alias.
3. Open **Account** / **General** settings and copy the **Public Key**.
4. In EmailJS account security settings, restrict allowed origins/domains to `https://nanethlucana.github.io` if that option is available. The full site URL is `https://nanethlucana.github.io/PINOYANA-Web/`; the allowed origin must not include the repository path.

## Create the order email template

1. Under **Email Templates**, create a template and set **To Email** to `pinoyana2026@gmail.com`. Keep the recipient fixed in the template rather than accepting a recipient from the browser.
2. Set the subject to `PINOYANA – New Order Received!`.
3. Use a body such as:

   ```text
   A customer has submitted a new order.

   Customer Name: {{customer_name}}
   Customer Contact Number: {{customer_contact}}
   Selected Product: {{product}}
   Quantity: {{quantity}}
   Preference: {{preference}}
   Additional Notes: {{additional_notes}}
   Total Amount: {{total_amount}}
   ```

4. Save the template and copy its **Template ID**. The template variables must use the exact spellings shown above.

## Create the feedback email template

1. Create a second template and set **To Email** to `pinoyana2026@gmail.com`.
2. Set the subject to `PINOYANA – New Customer Feedback!`.
3. Use a body such as:

   ```text
   A customer has submitted feedback.

   Customer Name: {{customer_name}}
   Product Rating: {{product_rating}}
   Customer Comments: {{customer_comments}}
   ```

4. Save the template and copy its **Template ID**. The template variables must use the exact spellings shown above.

## Configure and publish the site

`index.html` is configured with the supplied EmailJS values:

- Public Key: `QTOl8tDo3oTlWDQTA`
- Service ID: `service_0fri38o`
- Order Template ID: `template_iojuv8s`
- Feedback Template ID: `template_nxp52t2`

The website uses the exact Gmail Service ID `service_0fri38o`. Keep each template's **To Email** fixed to `pinoyana2026@gmail.com`. The order template variables expected by the website are `customer_name`, `customer_contact`, `product`, `quantity`, `preference`, `additional_notes`, and `total_amount`. The feedback template variables are `customer_name`, `product_rating`, and `customer_comments`. Publish the site through GitHub Pages and test each form with real submissions. The existing on-site confirmation appears only when EmailJS's send request succeeds. Check the recipient Gmail inbox (including Spam) and the EmailJS dashboard for send status before considering delivery verified.

## Diagnose a failed submission

The configured website origin is `https://nanethlucana.github.io` (the site path `/PINOYANA-Web/` is not part of the origin). Ensure the current `index.html` is published to the `PINOYANA-Web` repository before testing.

The page initializes the SDK before registering form handlers and logs only request type, public service/template IDs, and provider status/message; customer form values are not logged. After publishing, open browser DevTools:

1. In **Network**, submit an order and feedback once each. Expect a `POST` to `https://api.emailjs.com/api/v1.0/email/send` for each submission.
2. In **Console**, inspect `[EmailJS]` diagnostics. A missing SDK is logged during page initialization. Rejected requests log their HTTP status and a redacted provider message.
3. If a request reaches EmailJS but is rejected, verify the Public Key and exact Service ID `service_0fri38o`, the two template IDs, each template's fixed **To Email** of `pinoyana2026@gmail.com`, and that all template variables exactly match the lists above.
4. Treat an EmailJS success response as provider acceptance, not proof of inbox delivery; check Email History and the recipient inbox including Spam.

## Duplicate and delivery limitations

The forms disable the submit button while a request is in progress and suppress a repeated identical submission in the current page session. EmailJS browser sending does not provide the same server-side idempotency guarantee as the previous backend design: retries after a network timeout and submissions from a reloaded page may send duplicates. EmailJS success means the provider accepted the request; it does not by itself prove inbox delivery. If guaranteed deduplication or server-verified delivery is required, use a backend with durable idempotency storage and provider delivery-status handling.
