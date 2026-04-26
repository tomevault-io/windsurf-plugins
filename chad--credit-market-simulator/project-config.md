---
trigger: always_on
description: use the following as a reference plus search the web for your own information.  I want an interactive web-based credit simulatory that models the real world and helps the user learn how the credit system in the US works.
---

use the following as a reference plus search the web for your own information.  I want an interactive web-based credit simulatory that models the real world and helps the user learn how the credit system in the US works.






// credit_system.ts
// A tiny, self-contained model of the U.S. consumer credit market for developers.
// No dependencies. Run with: `ts-node credit_system.ts` (or compile with tsc).

/* =========================
 * 1) Shared Types & Utils
 * ========================= */

type Money = number; // dollars; keep it simple

const round2 = (n: number) => Math.round(n * 100) / 100;

// Deterministic PRNG for repeatable sims
class PRNG {
  private s = 42;
  rand() {
    // xorshift-ish
    let x = this.s ^= this.s << 13;
    x ^= x >> 17;
    x ^= x << 5;
    this.s = x;
    return Math.abs(x % 10000) / 10000;
  }
}

/* =========================
 * 2) Domain Entities
 * ========================= */

// Borrower/Consumer
interface BorrowerProfile {
  id: string;
  name: string;
  monthlyIncome: Money;
  monthlyExpenses: Money;
  cashOnHand: Money;
}

type TradelineType = "CreditCard" | "PersonalLoan" | "Auto" | "Mortgage" | "BNPL";

interface TradelineSnapshot {
  lenderId: string;
  type: TradelineType;
  openedAtMonth: number;
  creditLimit?: Money;
  principalOriginal: Money;
  principalCurrent: Money;
  status: "Current" | "Late30" | "Late60" | "Defaulted" | "Closed";
}

interface BureauRecord {
  borrowerId: string;
  score: number; // FICO-like 300-850
  tradelines: TradelineSnapshot[];
  lastUpdatedMonth: number;
}

// Simple loan
interface LoanTerms {
  apr: number; // e.g., 0.24 = 24% APR
  termMonths: number;
  originationFeePct: number; // taken up-front
  type: TradelineType;
}

interface Loan {
  id: string;
  borrowerId: string;
  originatorId: string;
  terms: LoanTerms;
  principalOriginal: Money;
  principalCurrent: Money;
  monthlyPayment: Money; // fixed payment for simplicity
  status: "Active" | "Defaulted" | "PaidOff" | "Sold";
  openedAtMonth: number;
}

// Lenders (Originators) & Servicers
interface OriginatorConfig {
  id: string;
  name: string;
  targetROA: number; // baked into pricing/fees
  servicingPartnerId: string;
}

interface ServicerConfig {
  id: string;
  name: string;
  annualServicingFeePct: number; // extracted from cash flows
  daysToReportToBureau: number; // latency
}

// Investors & Securitization
interface Security {
  id: string;
  name: string;
  collateralLoanIds: string[];
  couponAPR: number; // what the security promises to pay investors
  servicerId: string;
  creationMonth: number;
  status: "Active" | "Matured";
  poolPrincipalCurrent: Money;
}

interface InvestorPosition {
  investorId: string;
  securityId: string;
  invested: Money;
  accruedInterest: Money;
}

interface Investor {
  id: string;
  name: string;
  costOfCapitalAPR: number; // hurdle/expectation
  cash: Money;
  positions: InvestorPosition[];
}

// Regulators (modeled as constraints/validators)
type RegulatoryEvent =
  | { type: "ECOA_NoticeLogged"; borrowerId: string; originatorId: string; month: number }
  | { type: "FCRA_AdverseActionLogged"; borrowerId: string; originatorId: string; month: number; reason: string }
  | { type: "TILA_DisclosureProvided"; borrowerId: string; originatorId: string; apr: number; month: number };

class Regulator {
  events: RegulatoryEvent[] = [];
  log(e: RegulatoryEvent) {
    this.events.push(e);
  }
}

/* =========================
 * 3) Messaging & Eventual Consistency
 * ========================= */

type DomainEvent =
  | { type: "CreditApplied"; borrowerId: string; month: number }
  | { type: "CreditDecisioned"; borrowerId: string; originatorId: string; loanId: string; approved: boolean; apr: number; month: number }
  | { type: "FundsDisbursed"; loanId: string; amount: Money; feesTaken: Money; month: number }
  | { type: "PaymentPosted"; loanId: string; borrowerId: string; amount: Money; interest: Money; principal: Money; servicerId: string; month: number }
  | { type: "DelinquencyUpdated"; loanId: string; status: TradelineSnapshot["status"]; month: number }
  | { type: "BureauReportReady"; borrowerId: string; tradeline: TradelineSnapshot; scheduledMonth: number }
  | { type: "BureauUpdated"; borrowerId: string; score: number; month: number }
  | { type: "LoansSecuritized"; securityId: string; loanIds: string[]; couponAPR: number; month: number }
  | { type: "InvestorSubscribed"; investorId: string; securityId: string; amount: Money; month: number }
  | { type: "SecurityDistribution"; securityId: string; investorId: string; interest: Money; principal: Money; month: number };

class EventBus {
  private queue: { deliverAtMonth: number; event: DomainEvent }[] = [];
  subscribe(fn: (e: DomainEvent) => void) {
    this.handlers.push(fn);
  }
  schedule(event: DomainEvent, deliverAtMonth: number) {
    this.queue.push({ deliverAtMonth, event });
  }
  tick(currentMonth: number) {
    const due = this.queue.filter(x => x.deliverAtMonth <= currentMonth);
    this.queue = this.queue.filter(x => x.deliverAtMonth > currentMonth);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
