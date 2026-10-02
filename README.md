"""
JK Pvt Ltd - Annual Financial Report FY 2025-26
All amounts in INR lakhs. Figures are ILLUSTRATIVE sample data:
edit the dictionaries below with real numbers and re-run.
"""

COMPANY = "JK Pvt Ltd"
PERIOD = "FY 2025-26 (1 Apr 2025 - 31 Mar 2026)"
TAX_RATE = 0.2517

# ---------------- INPUTS ----------------
# Profit & Loss inputs: current year and previous year
pl_input = {
    "FY26": dict(revenue=4850, cogs=2910, employee=620, admin=340, depreciation=130, finance=120, tax=184),
    "FY25": dict(revenue=4200, cogs=2583, employee=560, admin=310, depreciation=115, finance=130, tax=126),
}

quarterly_revenue = {"Q1": 1090, "Q2": 1180, "Q3": 1260, "Q4": 1320}

assets = {
    "Net fixed assets": 1450,
    "Inventory": 620,
    "Trade receivables": 780,
    "Cash & bank": 410,
    "Other current assets": 140,
}
liabilities = {
    "Share capital": 500,
    "Reserves & surplus": 1350,
    "Long-term borrowings": 700,
    "Trade payables": 540,
    "Other current liabilities": 310,
}

cash_flow = {
    "Cash flow from operations": 640,
    "Capital expenditure": -310,
    "Repayment of borrowings": -180,
    "Interest paid": -120,
}
opening_cash = 380


# ---------------- CALCULATIONS ----------------
def build_pl(d):
    r = dict(d)
    r["gross_profit"] = d["revenue"] - d["cogs"]
    r["ebitda"] = r["gross_profit"] - d["employee"] - d["admin"]
    r["ebit"] = r["ebitda"] - d["depreciation"]
    r["pbt"] = r["ebit"] - d["finance"]
    r["pat"] = r["pbt"] - d["tax"]
    return r


def pct_change(new, old):
    return (new - old) / old * 100


def money(x):
    return f"({abs(x):,.0f})" if x < 0 else f"{x:,.0f}"


def line(label, *cols, width=34):
    return f"{label:<{width}}" + "".join(f"{c:>14}" for c in cols)


def heading(title):
    print("\n" + "=" * 62)
    print(title)
    print("=" * 62)


def main():
    cur, prev = build_pl(pl_input["FY26"]), build_pl(pl_input["FY25"])

    print(f"{COMPANY.upper()} - ANNUAL FINANCIAL REPORT")
    print(PERIOD)
    print("Amounts in INR lakhs | Illustrative data, unaudited")

    # 1. Profit & Loss
    heading("1. PROFIT & LOSS STATEMENT")
    print(line("Particulars", "FY 2025-26", "FY 2024-25", "Change %"))
    print("-" * 76)
    rows = [
        ("Revenue from operations", "revenue", 1),
        ("Cost of goods sold", "cogs", -1),
        ("GROSS PROFIT", "gross_profit", 1),
        ("Employee costs", "employee", -1),
        ("Admin & selling expenses", "admin", -1),
        ("EBITDA", "ebitda", 1),
        ("Depreciation", "depreciation", -1),
        ("EBIT", "ebit", 1),
        ("Finance costs", "finance", -1),
        ("PROFIT BEFORE TAX", "pbt", 1),
        ("Tax", "tax", -1),
        ("PROFIT AFTER TAX", "pat", 1),
    ]
    for label, key, sign in rows:
        print(line(label, money(sign * cur[key]), money(sign * prev[key]),
                   f"{pct_change(cur[key], prev[key]):+.1f}%"))

    print("\nQuarterly revenue:")
    for q, v in quarterly_revenue.items():
        print(f"  {q}: {v:,}")
    assert sum(quarterly_revenue.values()) == cur["revenue"], "Quarterly revenue must sum to annual"

    # 2. Balance sheet
    heading("2. BALANCE SHEET (as at 31 March 2026)")
    total_assets, total_liab = sum(assets.values()), sum(liabilities.values())
    print("ASSETS")
    for k, v in assets.items():
        print(line("  " + k, money(v)))
    print(line("TOTAL ASSETS", money(total_assets)))
    print("\nEQUITY & LIABILITIES")
    for k, v in liabilities.items():
        print(line("  " + k, money(v)))
    print(line("TOTAL EQUITY & LIABILITIES", money(total_liab)))
    print("Balanced:", "YES" if total_assets == total_liab else "NO - check inputs")

    # 3. Cash flow
    heading("3. CASH FLOW SUMMARY")
    for k, v in cash_flow.items():
        print(line(k, money(v)))
    fcf = cash_flow["Cash flow from operations"] + cash_flow["Capital expenditure"]
    net_change = sum(cash_flow.values())
    closing_cash = opening_cash + net_change
    print(line("FREE CASH FLOW", money(fcf)))
    print(line("Net increase in cash", money(net_change)))
    print(line("Opening cash", money(opening_cash)))
    print(line("Closing cash", money(closing_cash)))
    if closing_cash != assets["Cash & bank"]:
        print("WARNING: closing cash does not match balance sheet cash")

    # 4. Ratios
    heading("4. KEY FINANCIAL RATIOS")
    equity = liabilities["Share capital"] + liabilities["Reserves & surplus"]
    debt = liabilities["Long-term borrowings"]
    cur_assets = sum(assets[k] for k in ["Inventory", "Trade receivables", "Cash & bank", "Other current assets"])
    cur_liab = liabilities["Trade payables"] + liabilities["Other current liabilities"]

    rec_days = assets["Trade receivables"] / cur["revenue"] * 365
    inv_days = assets["Inventory"] / cur["cogs"] * 365
    pay_days = liabilities["Trade payables"] / cur["cogs"] * 365

    ratios = [
        ("Gross margin", f"{cur['gross_profit'] / cur['revenue']:.1%}"),
        ("EBITDA margin", f"{cur['ebitda'] / cur['revenue']:.1%}"),
        ("Net profit margin", f"{cur['pat'] / cur['revenue']:.1%}"),
        ("Return on equity", f"{cur['pat'] / equity:.1%}"),
        ("Current ratio", f"{cur_assets / cur_liab:.2f}x"),
        ("Quick ratio", f"{(cur_assets - assets['Inventory']) / cur_liab:.2f}x"),
        ("Debt-to-equity", f"{debt / equity:.2f}x"),
        ("Net debt / EBITDA", f"{(debt - assets['Cash & bank']) / cur['ebitda']:.2f}x"),
        ("Interest coverage (EBIT/interest)", f"{cur['ebit'] / cur['finance']:.1f}x"),
        ("Receivable days", f"{rec_days:.0f}"),
        ("Inventory days", f"{inv_days:.0f}"),
        ("Payable days", f"{pay_days:.0f}"),
        ("Cash conversion cycle (days)", f"{rec_days + inv_days - pay_days:.0f}"),
    ]
    for name, val in ratios:
        print(line(name, val))

    # 5. Observations
    heading("5. OBSERVATIONS & RECOMMENDATIONS")
    growth = pct_change(cur["revenue"], prev["revenue"])
    profit_growth = pct_change(cur["pat"], prev["pat"])
    print(f"- Revenue grew {growth:.1f}% while profit grew {profit_growth:.1f}%: strong operating leverage.")
    print(f"- Leverage is low (D/E {debt / equity:.2f}x); interest is covered {cur['ebit'] / cur['finance']:.1f} times.")
    print(f"- Working capital is the main drag (cash cycle {rec_days + inv_days - pay_days:.0f} days).")
    release = (rec_days - 50) / 365 * cur["revenue"]
    print(f"- Cutting receivable days to 50 would release about INR {release:,.0f} lakhs.")
    print("- Move to monthly reporting with a rolling 12-month cash forecast.")


if __name__ == "__main__":
    main()
