---
layout: slide
title: "Loan Amount Selection Mockup (MXN)"
---

<div class="loan-mockup">
  <p class="subtitle">Prototype UI for a loan amount selector with fixed installment payments.</p>

  <div class="loan-card">
    <div class="header-row">
      <span class="label">Max credit limit</span>
      <strong id="maxCreditText">MXN $120,000.00</strong>
    </div>

    <div class="control-group">
      <div class="row">
        <label for="loanAmount">Loan amount (MXN)</label>
        <strong id="loanAmountText">MXN $30,000.00</strong>
      </div>
      <input id="loanAmount" type="range" min="1000" max="120000" step="500" value="30000" />
      <div class="range-legend">
        <span>MXN $1,000</span>
        <span>MXN $120,000</span>
      </div>
    </div>

    <div class="control-group">
      <div class="row">
        <label for="dailyRate">Daily interest rate</label>
        <strong id="dailyRateText">0.30%</strong>
      </div>
      <input id="dailyRate" type="range" min="0.2" max="0.5" step="0.01" value="0.3" />
      <div class="range-legend">
        <span>0.20% daily</span>
        <span>0.50% daily</span>
      </div>
    </div>

    <div class="control-group">
      <div class="row">
        <span>Term (installments)</span>
      </div>
      <div id="termOptions" class="term-options"></div>
    </div>

    <div class="results">
      <div class="result-item">
        <span>Fixed payment at each due date</span>
        <strong id="installmentText">MXN $0.00</strong>
      </div>
      <div class="result-item">
        <span>Total repayment</span>
        <strong id="totalText">MXN $0.00</strong>
      </div>
      <div class="result-item">
        <span>Total interest (simple interest)</span>
        <strong id="interestText">MXN $0.00</strong>
      </div>
      <div class="result-item">
        <span>Selected term and due cadence</span>
        <strong id="termSummary">6 installments, every 30 days</strong>
      </div>
    </div>
  </div>
</div>

<style>
  .loan-mockup {
    max-width: 860px;
    margin: 0 auto;
    text-align: left;
  }

  .loan-mockup .subtitle {
    margin-bottom: 0.8rem;
    color: #9fb3c8;
    font-size: 1rem;
  }

  .loan-mockup .loan-card {
    border: 1px solid rgba(148, 163, 184, 0.35);
    border-radius: 16px;
    padding: 1.1rem;
    background: rgba(15, 23, 42, 0.82);
    box-shadow: 0 10px 24px rgba(0, 0, 0, 0.24);
  }

  .loan-mockup .header-row,
  .loan-mockup .row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
  }

  .loan-mockup .header-row {
    padding: 0.5rem 0.1rem 0.9rem;
    margin-bottom: 0.8rem;
    border-bottom: 1px solid rgba(148, 163, 184, 0.25);
  }

  .loan-mockup .label {
    color: #9fb3c8;
  }

  .loan-mockup .control-group {
    margin-bottom: 1rem;
  }

  .loan-mockup label,
  .loan-mockup .row span {
    font-size: 0.95rem;
    color: #d5e2ef;
  }

  .loan-mockup input[type="range"] {
    width: 100%;
    accent-color: #22d3ee;
    margin: 0.35rem 0 0.2rem;
  }

  .loan-mockup .range-legend {
    display: flex;
    justify-content: space-between;
    color: #9fb3c8;
    font-size: 0.78rem;
  }

  .loan-mockup .term-options {
    display: grid;
    grid-template-columns: repeat(6, minmax(0, 1fr));
    gap: 0.45rem;
    margin-top: 0.45rem;
  }

  .loan-mockup .term-button {
    border: 1px solid rgba(148, 163, 184, 0.35);
    border-radius: 9px;
    background: rgba(30, 41, 59, 0.65);
    color: #e2e8f0;
    padding: 0.35rem 0.25rem;
    font-size: 0.76rem;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .loan-mockup .term-button:hover {
    border-color: rgba(34, 211, 238, 0.7);
  }

  .loan-mockup .term-button.active {
    background: rgba(34, 211, 238, 0.2);
    border-color: #22d3ee;
    color: #ecfeff;
    font-weight: 700;
  }

  .loan-mockup .results {
    margin-top: 0.8rem;
    border-top: 1px solid rgba(148, 163, 184, 0.25);
    padding-top: 0.85rem;
    display: grid;
    gap: 0.5rem;
  }

  .loan-mockup .result-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 0.75rem;
    font-size: 0.88rem;
  }

  .loan-mockup .result-item span {
    color: #9fb3c8;
  }

  .loan-mockup .result-item strong {
    color: #f8fafc;
  }
</style>

<script>
  (function () {
    var MAX_CREDIT = 120000;
    var DAYS_PER_INSTALLMENT = 30;
    var selectedTerm = 6;

    var amountInput = document.getElementById("loanAmount");
    var rateInput = document.getElementById("dailyRate");
    var termOptions = document.getElementById("termOptions");

    if (!amountInput || !rateInput || !termOptions) {
      return;
    }

    var amountText = document.getElementById("loanAmountText");
    var rateText = document.getElementById("dailyRateText");
    var maxCreditText = document.getElementById("maxCreditText");
    var installmentText = document.getElementById("installmentText");
    var totalText = document.getElementById("totalText");
    var interestText = document.getElementById("interestText");
    var termSummary = document.getElementById("termSummary");

    var mxnFormatter = new Intl.NumberFormat("es-MX", {
      style: "currency",
      currency: "MXN",
      minimumFractionDigits: 2,
      maximumFractionDigits: 2
    });

    function formatMXN(value) {
      return mxnFormatter.format(value);
    }

    function createTermButtons() {
      var i;
      for (i = 1; i <= 12; i += 1) {
        var button = document.createElement("button");
        button.type = "button";
        button.className = "term-button";
        button.setAttribute("data-term", i);
        button.textContent = i + " mo";
        if (i === selectedTerm) {
          button.classList.add("active");
        }
        button.addEventListener("click", function (event) {
          selectedTerm = parseInt(event.currentTarget.getAttribute("data-term"), 10);
          update();
        });
        termOptions.appendChild(button);
      }
    }

    function setActiveTermButton() {
      var buttons = termOptions.querySelectorAll(".term-button");
      var i;
      for (i = 0; i < buttons.length; i += 1) {
        var buttonTerm = parseInt(buttons[i].getAttribute("data-term"), 10);
        buttons[i].classList.toggle("active", buttonTerm === selectedTerm);
      }
    }

    function update() {
      var amount = Number(amountInput.value);
      var dailyRatePercent = Number(rateInput.value);
      var totalDays = selectedTerm * DAYS_PER_INSTALLMENT;
      var totalInterest = amount * (dailyRatePercent / 100) * totalDays;
      var totalRepayment = amount + totalInterest;
      var fixedInstallment = totalRepayment / selectedTerm;

      maxCreditText.textContent = formatMXN(MAX_CREDIT);
      amountText.textContent = formatMXN(amount);
      rateText.textContent = dailyRatePercent.toFixed(2) + "%";
      installmentText.textContent = formatMXN(fixedInstallment);
      totalText.textContent = formatMXN(totalRepayment);
      interestText.textContent = formatMXN(totalInterest);
      termSummary.textContent = selectedTerm + " installments, every " + DAYS_PER_INSTALLMENT + " days";
      setActiveTermButton();
    }

    amountInput.addEventListener("input", update);
    rateInput.addEventListener("input", update);

    createTermButtons();
    update();
  })();
</script>
