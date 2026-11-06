# Company Policies

> **ruby-veterinary RAG Document**
>
> **Classification:** public-safe
>
> **Purpose:** Customer-facing company policies for AI-assisted question answering.
>
> **Last Updated:** 2026-11-06

---

## 1. Cancellation Policy

### Appointment Cancellations

- Appointments may be cancelled or rescheduled up to **24 hours** before the scheduled time without charge.
- Cancellations made with less than 24 hours' notice may be subject to a late-cancellation fee at the clinic's discretion.
- No-shows — appointments missed without any cancellation — may be charged the full consultation fee and may affect future booking availability.
- To cancel or reschedule, call the clinic directly or use the appointment request section on the website. If you reach us after hours, leave a message and we will confirm the cancellation on the next business day.

### Order Cancellations

- Orders that have not yet been fulfilled (status `pending_payment` or `awaiting_prescription`) can be cancelled on request. Contact the clinic by phone to request a cancellation.
- Once an order has moved to `fulfilling` or `fulfilled`, it cannot be cancelled. If the order has already shipped, please see the Refund Policy section below.
- Prescription orders that are awaiting veterinary authorisation can be cancelled at any point before approval.

### Subscription Cancellations

- Subscriptions for auto-refill products (preventatives, therapeutic diets, wellness items) can be paused or cancelled at any time.
- Cancelling a subscription stops all future orders. Existing orders that have already been placed are unaffected and will be fulfilled as normal.
- Pausing a subscription suspends the next scheduled order without cancelling the subscription. You can resume at any time.
- You can manage your subscriptions through your account on the website or by contacting the clinic.

---

## 2. Payment Policy

### Accepted Payment Methods

- Payments are processed through secure, tokenised third-party gateways (such as Stripe or PayPal). The clinic never stores or has access to your full card number, CVV, or expiry date.
- Accepted payment methods depend on the gateway selected at checkout. Typically, major credit cards, debit cards, and digital wallets are supported.

### When Payment Is Captured

- Payment is authorised at checkout and **captured** when the order is confirmed.
- For orders containing **prescription items**, payment may be authorised at checkout but the order remains on hold until a veterinarian approves the prescription. Once approved, the payment is captured and the order proceeds to fulfilment. If the prescription is rejected, the payment is refunded.
- Subscriptions are billed automatically on the scheduled order date. You will be notified before each billing cycle.

### Payment Security

- All transactions occur over HTTPS/TLS.
- The clinic is PCI-DSS compliant through its tokenised payment gateway. Card data never touches clinic servers.
- Every checkout is **idempotent**, meaning that if a network issue occurs and you try again, you will never be charged twice for the same order.

### Invoices and Receipts

- A receipt is emailed to you after every successful payment. You can also view your order history and download invoices from your account on the website.

---

## 3. Refund Policy

### When Refunds Are Issued

- **Prescription rejected:** If a veterinarian declines to authorise a prescription order, any captured payment is refunded in full.
- **Order not fulfilled:** If an order cannot be fulfilled due to stock unavailability or other clinic-side reasons, a full refund is issued.
- **Damaged or incorrect items:** If you receive a damaged product or an item different from what you ordered, contact the clinic within **7 days** of delivery. We will arrange a return, replacement, or full refund.
- **Change of mind:** Unopened, non-prescription products in resalable condition may be returned within **14 days** of purchase for a refund. Prescription medication and therapeutic diets cannot be returned once dispensed, for safety and regulatory reasons.
- **Digital products:** Downloadable products (care sheets, certificates) are generally non-refundable once downloaded. Contact the clinic if you experience technical issues accessing your download.

### Refund Process

- Refunds are processed back to the original payment method used at checkout.
- Processing time depends on your payment provider and typically takes **5–10 business days** to appear on your statement.
- To request a refund, call the clinic or speak with reception. Please have your order number ready.

### Partial Refunds

- If only part of an order is returned or unavailable, a partial refund is issued for the affected items. Shipping costs are not refunded unless the return is due to clinic error or damage.

---

## 4. Shipping & Pickup Policy

### Fulfilment Options

When you place an order, you can choose from three fulfilment methods:

| Method | Description |
|--------|-------------|
| **Ship to door** | Your order is shipped to the address on file. Shipping costs are calculated at checkout based on weight and destination. |
| **Free in-clinic pickup** | Collect your order from the clinic during business hours at no additional cost. You will be notified when your order is ready for pickup. |
| **Digital delivery** | Downloadable products (care sheets, certificates) are delivered instantly after payment. You will receive a secure download link. |

### Shipping Timelines

- In-stock items are typically dispatched within **1–2 business days** of order confirmation.
- Prescription items require veterinary authorisation before dispatch. Please allow **up to one business day** for the authorisation review. Once approved, the order moves to fulfilment.
- Estimated delivery times depend on your location and the shipping method selected. Tracking information is provided once your order ships.

### In-Clinic Pickup

- You will receive a notification (email or WhatsApp) when your order is ready for collection.
- Bring your order confirmation and a form of ID when you arrive.
- Pickup is available during regular business hours. Check the clinic's hours page or call ahead to confirm.
- If you cannot collect within **7 days** of notification, please contact the clinic to make alternative arrangements.

### Shipping Restrictions

- Prescription medication can only be shipped within the country where the clinic is licensed to operate.
- Some heavy or bulky items may have limited shipping options.

---

## 5. Vaccination Requirements

### For Clinic Visits

- All pets visiting the clinic for routine appointments, surgeries, or boarding must be up to date on their core vaccinations.
- **Dogs:** Rabies, DHPP (Distemper, Hepatitis, Parvovirus, Parainfluenza). Bordetella (kennel cough) is recommended for dogs that board or socialise with other dogs.
- **Cats:** Rabies, FVRCP (Feline Viral Rhinotracheitis, Calicivirus, Panleukopenia). Feline Leukaemia (FeLV) is recommended for outdoor cats.
- **Rabbits:** Myxomatosis and RHD (Rabbit Haemorrhagic Disease) vaccinations.
- Please bring your pet's vaccination record or card to every visit. If records are unavailable, the veterinarian may recommend restarting or titre-testing before proceeding with certain procedures.

### For Prescription Dispensing

- A valid veterinarian-client-patient relationship is required before any prescription medication is dispensed. This means your pet must have been examined by one of our veterinarians within a timeframe appropriate to the condition being treated.
- Prescription orders placed through the website will be held until a veterinarian reviews your pet's record and authorises the order.

### Puppy and Kitten Vaccination Schedules

- **Puppies:** Typically start vaccinations at 6–8 weeks, with boosters every 3–4 weeks until 16 weeks of age.
- **Kittens:** Typically start at 8–9 weeks, with boosters every 3–4 weeks until 16 weeks of age.
- A final booster at 12 months is recommended, followed by regular boosters as advised by your veterinarian based on your pet's lifestyle and risk factors.

---

## 6. Prescription Policy

### How Prescription Orders Work

Our online store sells three categories of products:

1. **General Pet Supplies** — no prescription needed. These items can be purchased freely.
2. **Therapeutic Diets** — may require a prescription depending on the specific formulation and your pet's health condition.
3. **Prescription Medication** — always requires veterinary authorisation.

When you order a prescription item, here is what happens:

1. You add the item to your cart and proceed to checkout.
2. At checkout, you must associate the item with one of your pets on file.
3. Your order is placed and enters an **"Awaiting Prescription"** hold. This is normal and expected — it is not an error.
4. A veterinarian reviews your order alongside your pet's medical record and the prescribing veterinarian's recommendation.
5. The veterinarian either **approves** the prescription (order proceeds to payment capture and fulfilment) or **rejects** it (you are notified and any payment is refunded).

### Turnaround Time

- Prescription reviews are typically completed within **one business day**.
- If the veterinarian has questions about the order, they may contact you for additional information, which may extend the review time.

### Why Prescription Approval Is Required

This process exists for your pet's safety. Prescription medications are powerful treatments that must be matched to the right patient, at the right dose, under the supervision of a veterinarian who knows your pet's history. A blocked checkout is always safer than a prescription dispensed to the wrong patient.

---

## 7. Privacy & Data Policy

### Data Protection Principles

- All traffic between your browser and our website is encrypted with **TLS/HTTPS**.
- **Payment card data** is never stored on or processed by clinic servers. All payment handling is delegated to a PCI-DSS compliant, tokenised third-party gateway.
- **Medical conversations** with the clinic through WhatsApp occur only over a verified WhatsApp Business API account. Personal health information is never discussed over unverified channels.
- Client and pet data is protected by default. Access to medical records is restricted to authorised clinic staff with a legitimate clinical need.

### Cookie Consent

- Our website uses cookies for essential functionality (cart, login session) and analytics. A cookie consent banner is displayed on your first visit. You can manage your preferences at any time through the cookie settings link in the page footer.

### Data Sharing

- We do not sell, rent, or share your personal data with third parties for marketing purposes.
- Data is shared with service providers only as necessary to deliver our services (payment processors, email delivery, WhatsApp). These providers are contractually bound to handle your data securely and only for the purpose we specify.

### Your Rights

- You may request a copy of the personal data we hold about you at any time.
- You may request corrections to inaccurate data.
- You may request deletion of your data, subject to legal and clinical record-keeping requirements (see Data Retention Policy below).
- Contact the clinic to exercise any of these rights.

---

## 8. Data Retention Policy

### Client and Pet Records

- Active client and pet records are retained for the duration of the client relationship plus a minimum period required by local veterinary regulations for clinical record-keeping.
- When a client relationship ends, records are soft-deleted: they are marked as inactive and removed from active queries and public-facing systems, but retained for the legally required clinical record retention period.
- After the retention period expires, records are permanently purged from all systems and backups.

### Medical History Records

- Medical history records (vaccination cards, lab results, uploaded documents) are retained alongside the pet's record for the full clinical retention period.
- Records uploaded by pet owners during the intake process are treated with the same retention and security standards as clinic-generated records.

### Order and Payment History

- Order records and payment references (which contain no card data — only tokenised gateway references) are retained for the duration required by tax and accounting regulations.
- After the legal retention period, order records are archived and eventually purged.

### Audit Logs

- Administrative action logs (staff changes to pricing, content publishing, role changes) are retained in an append-only log. Retention is governed by clinic operational policy. Logs never contain passwords, payment card data, or authentication tokens.

### Backups

- Encrypted daily backups of all clinic data are stored off-site with a separate provider. Backups follow the same retention schedule as the primary data they protect.
- When records are purged from the live system, they are also purged from the backup rotation at the next cycle.

### Deletion Requests

- Clients may request deletion of their personal data. The clinic will comply to the fullest extent possible while meeting legal obligations for clinical record-keeping, tax records, and regulatory compliance. Where full deletion is not immediately possible, records are soft-deleted and restricted from active use until the retention period expires.

---

## Related Documents

- `product-catalog.md` — Product types, categories, and how to shop
- `pet-care-faqs.md` — Frequently asked questions about pet care
- `../vision.md` — Company vision and guiding principles
- `../prd.md` — Product requirements document