# Phase 4: The Booking Journey (Transactional)
*Date planning, payments, and booking redemption.*

### 1. Date Planning
- [ ] **Proposals:** `GET /stores/proximity`, `POST /dates/propose`, `POST /dates/respond`.
- [ ] **Reminders:** 24-hour automated notification before `match.date_time`.

### 2. Wallet & Checkout
- [ ] **Wallet:** `GET /wallet/transactions`, `POST /wallet/fund`.
- [ ] **Payment:** `POST /checkout`, `POST /payment/pay` - Supports split billing or full payment.
- [ ] **Commission:** Automated platform fee deduction via `public.process_order_payment`.

### 3. Redemption
- [ ] **Verification:** `POST /vendor/orders/verify` - Scanner integration for booking code redemption.
- [ ] **Escrow Release:** Auto-transfer from platform escrow wallet to vendor's operating wallet.
