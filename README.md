import requests
import time
import random

# Deriv API Configuration
API_URL = "https://api.deriv.com"
API_TOKEN = "your_api_token_here"  # Replace with your Deriv API token
ACCOUNT_TYPE = "virtual"  # Use "real" for real money account
SYMBOL = "R_100"  # Example: Volatility 100 Index
STAKE = 0.5  # Initial stake amount
MAX_LOSS = 30  # Maximum acceptable loss
TAKE_PROFIT = 10  # Take profit amount
VOLATILITY = 25  # Probability of losing a trade (in percentage)

# Headers for API requests
HEADERS = {
    "Authorization": f"Bearer {API_TOKEN}",
    "Content-Type": "application/json",
}

class DerivTradingBot:
    def __init__(self):
        self.current_balance = self.get_balance()
        self.initial_balance = self.current_balance
        self.current_stake = STAKE
        self.consecutive_losses = 0

    def get_balance(self):
        """Retrieve the current account balance."""
        response = requests.get(f"{API_URL}/balance", headers=HEADERS)
        if response.status_code == 200:
            return float(response.json()["balance"]["balance"])
        else:
            raise Exception(f"Failed to fetch balance: {response.text}")

    def place_trade(self):
        """Place a trade on Deriv."""
        payload = {
            "proposal": 1,
            "amount": self.current_stake,
            "basis": "stake",
            "contract_type": "DIGITDIFF",  # Example: Digits Difference
            "currency": "USD",
            "duration": 5,  # Duration in ticks
            "duration_unit": "t",
            "symbol": SYMBOL,
        }
        response = requests.post(f"{API_URL}/buy", headers=HEADERS, json=payload)
        if response.status_code == 200:
            return response.json()["buy"]
        else:
            raise Exception(f"Failed to place trade: {response.text}")

    def simulate_trade(self):
        """Simulate a trade outcome based on volatility."""
        return random.choices(['win', 'loss'], weights=[100 - VOLATILITY, VOLATILITY])[0]

    def run_bot(self):
        trade_count = 0
        while self.current_balance > 0 and self.current_balance < self.initial_balance + TAKE_PROFIT:
            trade_count += 1
            outcome = self.simulate_trade()

            if outcome == 'win':
                self.current_balance += self.current_stake
                self.current_stake = STAKE  # Reset stake to initial after a win
                self.consecutive_losses = 0
                print(f"Trade {trade_count}: Won! Balance: ${self.current_balance:.2f}")
            else:
                self.current_balance -= self.current_stake
                self.consecutive_losses += 1
                self.current_stake *= 2  # Double the stake after a loss
                print(f"Trade {trade_count}: Lost! Balance: ${self.current_balance:.2f}")

            # Check if max loss is reached
            if self.current_balance <= self.initial_balance - MAX_LOSS:
                print(f"Max loss of ${MAX_LOSS} reached. Stopping bot.")
                break

            # Check if take profit is reached
            if self.current_balance >= self.initial_balance + TAKE_PROFIT:
                print(f"Take profit of ${TAKE_PROFIT} reached. Stopping bot.")
                break

            time.sleep(1)  # Add a delay between trades to avoid rate limits

        print(f"Final Balance: ${self.current_balance:.2f}")

# Initialize and run the bot
bot = DerivTradingBot()
bot.run_bot()
