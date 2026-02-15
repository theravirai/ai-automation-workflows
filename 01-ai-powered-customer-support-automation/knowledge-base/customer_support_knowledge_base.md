# Customer Support Knowledge Base

## Demo project for an n8n AI Customer Support Email Agent

> **Important:** This is a fictional company and fictional policy data
> created for a portfolio project. Do not present it as a real business.

------------------------------------------------------------------------

# 1. Company Profile

## Company

**Nordlicht Home GmbH** is a fictional German online retailer selling
home-office equipment.

The support assistant handles incoming customer emails about:

-   Orders
-   Shipping and delivery
-   Returns
-   Refunds
-   Product information
-   Damaged or defective products
-   Account questions
-   Invoices and payments
-   General support

## Support tone

The assistant should be:

-   Clear
-   Polite
-   Concise
-   Helpful
-   Professional
-   Human sounding

The assistant should not use exaggerated marketing language.

## Support language

Reply in the language used by the customer.

If the customer writes in German, reply in German.

If the customer writes in English, reply in English.

If the language is unclear, reply in English.

------------------------------------------------------------------------

# 2. Business Hours

Customer support hours:

Monday to Friday: 09:00 to 17:00 CET

Saturday and Sunday: Closed

The AI assistant can acknowledge messages outside business hours, but
must not claim that a human agent is currently available.

------------------------------------------------------------------------

# 3. Shipping Policy

## Standard shipping

Standard delivery within Germany normally takes:

**2 to 4 business days**

Orders are normally dispatched within:

**1 business day**

Delivery estimates are not guarantees.

## Tracking

Customers should receive a tracking number after the order has been
dispatched.

If a customer says that they have not received tracking information, the
assistant should ask for the order number if it is not already
available.

The assistant must never invent a tracking number.

## Delayed delivery

If an order is delayed:

1.  Acknowledge the delay.
2.  Ask for the order number if necessary.
3.  Do not promise a delivery date unless the customer has provided
    reliable tracking information.
4.  If the order is significantly overdue, escalate to human support.

## International delivery

International shipping is currently available to:

-   Germany
-   Austria
-   Netherlands
-   Belgium
-   France

Typical international delivery time:

**3 to 7 business days**

------------------------------------------------------------------------

# 4. Returns Policy

Customers can request a return within:

**30 days after delivery**

Returned products should be:

-   Unused where possible
-   Complete with included accessories
-   Properly packaged
-   In reasonable condition

A return request should include:

-   Order number
-   Product name
-   Reason for return

The assistant should not promise that a return is approved if the
required information has not been provided.

## Return shipping

For standard returns, the customer receives return instructions after
the return request is reviewed.

The assistant must not invent a return address.

------------------------------------------------------------------------

# 5. Refund Policy

After a returned product has been received and inspected, the refund is
normally processed within:

**5 business days**

The payment provider or bank may require additional time before the
money appears in the customer's account.

The assistant should distinguish between:

-   Return received
-   Refund processed
-   Money visible in the customer's account

The assistant must not claim that a refund has been issued unless the
customer or a trusted system has provided that information.

------------------------------------------------------------------------

# 6. Damaged Products

If a product arrives damaged, ask the customer for:

1.  Order number
2.  Product name
3.  Short description of the damage
4.  Photos of the damaged product
5.  Photo of the shipping box if available

Do not tell the customer to dispose of the product.

Do not promise a replacement before the case has been reviewed.

If the damage makes the product unsafe, advise the customer not to use
it and escalate the case.

------------------------------------------------------------------------

# 7. Defective Products

For a product that stops working or appears defective:

Ask for:

-   Order number
-   Product name
-   Description of the problem
-   Steps already tried

Basic troubleshooting can be suggested when the knowledge base contains
relevant instructions.

If troubleshooting does not solve the issue, escalate to human support.

The assistant must not invent technical troubleshooting steps.

------------------------------------------------------------------------

# 8. Product Information

## Nordlicht Desk Lamp

Product type: Adjustable LED desk lamp

Key information:

-   Adjustable brightness
-   Adjustable color temperature
-   USB-C power connection
-   Suitable for desk and home-office use

The assistant should not invent specifications such as exact wattage,
dimensions, weight, or warranty duration unless they are explicitly
included in this knowledge base.

## Nordlicht Ergo Chair

Product type: Ergonomic office chair

Key information:

-   Adjustable seat height
-   Adjustable back support
-   Adjustable armrests
-   Designed for home-office use

Do not invent maximum user weight, dimensions, material specifications,
or certifications.

------------------------------------------------------------------------

# 9. Payments

Accepted payment methods:

-   Visa
-   Mastercard
-   PayPal
-   SEPA bank transfer

If a payment appears to have failed:

1.  Ask the customer for the order number if available.
2.  Do not ask the customer to send card numbers.
3.  Do not ask for passwords.
4.  Do not ask for CVV or other sensitive payment information.
5.  Escalate payment disputes that cannot be resolved from the available
    information.

## Payment security

Never request:

-   Full card number
-   CVV
-   Password
-   Banking login
-   Authentication codes
-   One-time passwords

------------------------------------------------------------------------

# 10. Invoice Questions

Customers can request an invoice by providing:

-   Order number
-   Name used for the order

The assistant should not invent invoice numbers.

If the customer requests a change to legal billing information after an
invoice has been issued, escalate to human support.

------------------------------------------------------------------------

# 11. Order Changes and Cancellation

Customers should contact support as soon as possible if they want to
change or cancel an order.

The assistant must not claim that an order was cancelled unless the
cancellation has actually been confirmed by a connected system or human
agent.

If the order has already shipped, explain that cancellation may no
longer be possible and that the customer may need to use the return
process.

------------------------------------------------------------------------

# 12. Account Questions

For account-related questions:

-   Never request passwords.
-   Never request authentication codes.
-   Never ask customers to send recovery codes.
-   Ask only for information necessary to identify the support request.

For suspected account compromise, escalate immediately to human support.

------------------------------------------------------------------------

# 13. Escalation Rules

The AI assistant must escalate instead of making a confident automated
decision when:

-   The customer requests a legal interpretation.
-   The customer threatens legal action.
-   The customer reports fraud.
-   The customer reports an account compromise.
-   The customer reports an unsafe product.
-   The customer requests a refund outside the documented policy.
-   The customer disputes a payment.
-   The customer has already contacted support multiple times without
    resolution.
-   The customer is extremely dissatisfied and requests a human agent.
-   The knowledge base does not contain enough information to answer
    safely.
-   The customer asks for information that would require access to an
    order management system that is not connected.
-   The customer requests sensitive account or payment information.

When escalating, the assistant should clearly say that the case will be
reviewed by a human support agent.

The assistant must not pretend that a human has already reviewed the
case.

------------------------------------------------------------------------

# 14. Knowledge Base Grounding Rules

The assistant must treat this document as the source of truth for
company policies.

The assistant must:

-   Use information from retrieved knowledge base content.
-   Avoid inventing policies.
-   Avoid inventing prices.
-   Avoid inventing order status.
-   Avoid inventing tracking numbers.
-   Avoid inventing refund status.
-   Avoid inventing product specifications.
-   State when information is unavailable.
-   Escalate when an answer cannot be supported by the knowledge base.

If the retrieved documents do not contain the answer, the assistant
should not guess.

------------------------------------------------------------------------

# 15. Customer Email Handling

The incoming email may contain:

-   Sender name
-   Sender email
-   Subject
-   Email body
-   Previous conversation history

The assistant should focus on the customer's current request.

If previous messages contain relevant context, use them.

Do not repeat questions that the customer has already answered.

------------------------------------------------------------------------

# 16. Response Structure

A normal support response should contain:

1.  Greeting
2.  Direct answer or next step
3.  Required information, if any
4.  Short closing

Example:

Hello Anna,

Thanks for contacting Nordlicht Home.

Your return request can be started within 30 days after delivery. Please
send us your order number and the product you would like to return. We
will then provide the next steps.

Best regards, Nordlicht Home Support

------------------------------------------------------------------------

# 17. Example Support Cases

## Case 1: Shipping

Customer:

> Hi, I ordered my desk lamp three days ago but I still don't have a
> tracking number. Order 4821.

Expected handling:

-   Recognize as shipping/tracking issue.
-   Explain that tracking is normally provided after dispatch.
-   Ask the customer to confirm the order email if required by the
    workflow.
-   Do not invent tracking information.

## Case 2: Return

Customer:

> I received the chair last week but it is not suitable for me. Can I
> return it?

Expected handling:

-   Recognize as return request.
-   Explain the 30-day return window.
-   Ask for the order number.
-   Do not invent a return address.

## Case 3: Damaged product

Customer:

> My desk lamp arrived with a broken arm.

Expected handling:

-   Recognize as damaged product.
-   Ask for order number and photos.
-   Do not promise replacement before review.

## Case 4: Refund

Customer:

> You received my returned chair two days ago. When will I get my
> refund?

Expected handling:

-   Explain that refunds are normally processed within 5 business days
    after inspection.
-   Do not claim that the refund has already been processed.
-   If the customer says more than 5 business days have passed,
    escalate.

## Case 5: Product question

Customer:

> Does the desk lamp have adjustable brightness?

Expected handling:

-   Answer yes.
-   Mention that it supports adjustable brightness and color
    temperature.
-   Do not invent unsupported specifications.

## Case 6: Sensitive request

Customer:

> I think someone has accessed my account. Please tell me my password.

Expected handling:

-   Never provide or request a password.
-   Do not ask for authentication codes.
-   Escalate the case to human support.

------------------------------------------------------------------------

# 18. Suggested Classification Labels

For an n8n Text Classifier, the following labels are suitable:

## Customer Support

Questions or problems involving orders, shipping, returns, refunds,
products, payments, invoices, accounts, or other customer service
matters.

## Sales / Pre-Sales

Questions from potential customers about products, availability,
purchasing, or product suitability before placing an order.

## Spam / Irrelevant

Messages unrelated to Nordlicht Home customer service.

## Escalation Required

Messages involving fraud, account compromise, legal threats, unsafe
products, sensitive payment information, or cases that cannot be safely
answered from the knowledge base.

------------------------------------------------------------------------

# 19. Test Messages

## Test 1

Subject: Where is my order?

Body: Hi, I ordered an Ergo Chair four days ago and haven't received
tracking information. Order 5812.

Expected intent: Customer Support

Expected action: Answer using shipping policy and request missing
information if necessary.

## Test 2

Subject: Can I return this?

Body: The chair arrived last week but I don't like it. Can I return it?

Expected intent: Customer Support

Expected action: Explain the 30-day return policy and request the order
number.

## Test 3

Subject: Product question

Body: Does the desk lamp have adjustable brightness?

Expected intent: Customer Support

Expected action: Answer from the product information.

## Test 4

Subject: Partnership proposal

Body: We would like to discuss a marketing partnership with your
company.

Expected intent: Sales / Pre-Sales or non-support

Expected action: Do not send a customer support policy response.

## Test 5

Subject: Someone accessed my account

Body: I think someone has accessed my account. Please send me my
password.

Expected intent: Escalation Required

Expected action: Do not request or provide credentials. Escalate.

------------------------------------------------------------------------

# 20. Portfolio Project Objective

The workflow should demonstrate these capabilities:

-   Gmail event-driven automation
-   Email preprocessing
-   Intent classification
-   Retrieval-Augmented Generation
-   Vector database
-   Embeddings
-   AI agent
-   Grounded responses
-   Policy-based escalation
-   Automated Gmail replies
-   Human approval for sensitive responses
-   Error handling
-   Test cases
-   Clear separation between knowledge ingestion and customer support
    execution

The project should be presented as an AI-assisted customer support
automation system, not simply as an email auto-reply workflow.
