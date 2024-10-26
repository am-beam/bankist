<script>
export default {
  data() {
    return {
      accounts: [
      {
        owner: "Mika Ella",
          movements: [200, 455.23, -306.5],
          interestRate: 1.2,
          pin: 1234,
           movementsDates: [
            "2019-11-18T21:31:17.178Z",
            "2019-12-23T07:42:02.383Z",
            "2020-01-28T09:15:04.904Z",
          ],
          currency: "EUR",
          locale: "pt-PT",},
        {
          owner: "Jonas Schmedtmann",
          movements: [200, 455.23, -306.5, 25000, -642.21, -133.9, 79.97, 1300],
          interestRate: 1.2,
          pin: 1111,
           movementsDates: [
            "2019-11-18T21:31:17.178Z",
            "2019-12-23T07:42:02.383Z",
            "2020-01-28T09:15:04.904Z",
            "2020-04-01T10:17:24.185Z",
            "2020-05-08T14:11:59.604Z",
            "2022-12-01T17:01:17.194Z",
            "2022-12-06T23:36:17.929Z",
            "2022-12-08T10:51:36.790Z",
          ],
          currency: "EUR",
          locale: "pt-PT",
        },
        {
          owner: "Alamin Mt",
          movements: [2500, 400, -260, 830, -4210, 2200, 4600, -70],
          interestRate: 1.8,
          pin: 2222,
          movementsDates: [
            "2019-11-03T13:15:33.035Z",
            "2019-11-25T09:48:16.867Z",
            "2019-12-16T06:04:23.907Z",
            "2020-01-12T14:18:46.235Z",
            "2020-02-25T16:33:06.386Z",
            "2020-04-11T14:43:26.374Z",
            "2020-06-15T18:49:59.371Z",
            "2020-07-30T12:01:20.894Z",
          ],
          currency: "BDT",
          locale: "en-US",
        },
        {
          owner: "Jessica Davis",
          movements: [5000, 3400, -150, -790, -3210, -1000, 8500, -30],
          interestRate: 1.5,
          pin: 3333,
          movementsDates: [
            "2019-11-01T13:15:33.035Z",
            "2019-11-30T09:48:16.867Z",
            "2019-12-25T06:04:23.907Z",
            "2020-01-25T14:18:46.235Z",
            "2020-02-05T16:33:06.386Z",
            "2020-04-10T14:43:26.374Z",
            "2020-06-25T18:49:59.371Z",
            "2020-07-26T12:01:20.894Z",
          ],
          currency: "USD",
          locale: "en-US",
        },
      ],
      currentAccount: null,
      loginUsername: '',
      loginPin: '',
      isLoggedIn: false,
      displayedMovements: [],
      timer: null,
      timerDuration: 300,
      transferTo: '',
      transferAmount: '',
      loanAmount: '',
      closeUsername: '',
      closePin: '',
      sorted: false,
      depositCount: 0,
      withdrawalCount: 0,
    };
  },
  computed: {
    welcomeMessage() {
      return this.isLoggedIn ? `Welcome back ${this.currentAccount.owner.split(" ")[0]}!` : 'Log in to get started';
    },
    formattedBalance() {
      return this.formatCurrency(this.calculateBalance());
    },
    formattedIncome() {
      return this.formatCurrency(this.calculateSummary('incomes'));
    },
    formattedOutgoings() {
      return this.formatCurrency(this.calculateSummary('outgoings'));
    },
    formattedInterest() {
      return this.formatCurrency(this.calculateInterest());
    },
    formattedDate() {
      return new Date().toLocaleDateString();
    },
    logoutTimer() {
      const minutes = String(Math.floor(this.timerDuration / 60)).padStart(2, '0');
      const seconds = String(this.timerDuration % 60).padStart(2, '0');
      return `${minutes}:${seconds}`;
    },
  },
  methods: {
     created() {
      this.createUserName(this.accounts);
    },
    createUserNames() {
      this.accounts.forEach((acc) => {
        acc.username = acc.owner
          .toLowerCase()
          .split(" ")
          .map((name) => name[0])
          .join("");
      });
      console.log("User names created:", this.accounts.map(acc => acc.username)); // Log usernames
    },

    login() {
      this.createUserNames();

      // Log input values
      console.log("Username input:", this.loginUsername);
      console.log("Pin input:", this.loginPin);

      this.currentAccount = this.accounts.find(
        (acc) => acc.username === this.loginUsername.trim().toLowerCase() && acc.pin === Number(this.loginPin)
      );

       if (this.currentAccount) {
          this.displayedMovements = this.currentAccount.movements.map((mov, i) => ({
            amount: mov,
            date: this.currentAccount.movementsDates[i],
          })).sort((a, b) => new Date(b.date) - new Date(a.date)); // Sort by date descending
          this.isLoggedIn = true;
          this.startLogoutTimer();
        console.log("Login successful!", this.currentAccount);
      } 
    },
    calculateBalance() {
      return this.currentAccount ? this.currentAccount.movements.reduce((acc, mov) => acc + mov, 0) : 0;
    },
    calculateSummary(type) {
      if (!this.currentAccount) return 0;
      if (type === 'incomes') {
        return this.currentAccount.movements.filter(mov => mov > 0).reduce((acc, mov) => acc + mov, 0);
      } else if (type === 'outgoings') {
        return Math.abs(this.currentAccount.movements.filter(mov => mov < 0).reduce((acc, mov) => acc + mov, 0));
      }
    },
    calculateInterest() {
      if (!this.currentAccount) return 0;
      return this.currentAccount.movements
        .filter(mov => mov > 0)
        .map(deposit => (deposit * this.currentAccount.interestRate) / 100)
        .filter(int => int >= 1)
        .reduce((acc, int) => acc + int, 0);
    },
    formatCurrency(value) {
      return new Intl.NumberFormat('en-US', { // Force 'en-US' locale
    style: 'currency',
    currency: this.currentAccount.currency,
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(value);
    },
    formatMovementDate(date) {
      const targetDate = new Date(date);
      // Format the date to MM/DD/YYYY
      const month = String(targetDate.getMonth() + 1).padStart(2, '0'); // Months are zero-indexed
      const day = String(targetDate.getDate()).padStart(2, '0');
      const year = targetDate.getFullYear();

      return `${month}/${day}/${year}`;
    },
    startLogoutTimer() {
      clearInterval(this.timer);
      this.timerDuration = 300;
      this.timer = setInterval(() => {
        if (this.timerDuration === 0) {
          clearInterval(this.timer);
          this.isLoggedIn = false;
        } else {
          this.timerDuration--;
        }
      }, 1000);
    },
    transfer() {
      const amount = Number(this.transferAmount);
      const receiverAcc = this.accounts.find(acc => acc.username === this.transferTo);
      
      if (
        amount > 0 &&
        receiverAcc &&
        this.calculateBalance() >= amount &&
        receiverAcc.username !== this.currentAccount.username
      ) {
        // Update movements
        this.currentAccount.movements.unshift(-amount);
        receiverAcc.movements.unshift(amount);
        
        // Update movement dates
        const currentDate = new Date().toISOString();
        this.currentAccount.movementsDates.unshift(currentDate);
        receiverAcc.movementsDates.unshift(currentDate);
        
        // Increment withdrawal and deposit counts
        this.withdrawalCount++;
        this.depositCount++; // Since the receiver account receives the amount
        
        // Clear input fields
        this.transferAmount = '';
        this.transferTo = '';
        
        // Update UI
        this.updateUI();
      }
    },
    requestLoan() {
      const amount = Number(this.loanAmount);
      
      if (
        amount > 0 &&
        this.currentAccount.movements.some(mov => mov >= (amount * 0.1))
      ) {
        setTimeout(() => {
          // Add loan amount to movements
          this.currentAccount.movements.unshift(amount);
          this.currentAccount.movementsDates.unshift(new Date().toISOString());
          
          // Clear input field
          this.loanAmount = '';
          
          // Update UI
          this.updateUI();
        }, 1000);
      }
    },
    closeAccount() {
      if (
        this.closeUsername === this.currentAccount.username &&
        Number(this.closePin) === this.currentAccount.pin
      ) {
        const index = this.accounts.findIndex(acc => acc.username === this.currentAccount.username);
        this.accounts.splice(index, 1);
        this.isLoggedIn = false;
      }
      this.closeUsername = '';
      this.closePin = '';
    },
    sortMovements() {
      if (!this.isLoggedIn) return; // Ensure the user is logged in

      if (!this.sorted) {
        // Sort by amount (highest to lowest)
        this.displayedMovements.sort((a, b) => b.amount - a.amount);
      } else {
        // Sort back to default (most recent to oldest)
        this.displayedMovements.sort((a, b) => new Date(b.date) - new Date(a.date));
      }
      
      this.sorted = !this.sorted; // Toggle the sorting state
    },

    updateUI() {
      // Update displayedMovements with current account movements
      this.displayedMovements = this.currentAccount.movements.map((amount, i) => ({
        amount,
        date: this.currentAccount.movementsDates[i],
      }));
    },
  }
};
</script>

<template>
 <div id="app">
    <nav>
      <p class="welcome">{{ welcomeMessage }}</p>
      <img src="logo.png" alt="Logo" class="logo" />
      <form class="login" @submit.prevent="login">
        <input
          type="text"
          placeholder="user"
          v-model="loginUsername"
          class="login__input login__input--user"
        />
        <input
          type="password"
          placeholder="PIN"
          maxlength="4"
          v-model="loginPin"
          class="login__input login__input--pin"
        />
        <button class="login__btn">&rarr;</button>
      </form>
    </nav>

    <main class="app" v-if="isLoggedIn">
      <div class="balance">
        <div>
          <p class="balance__label">Current balance</p>
          <p class="balance__date">
            As of <span class="date">{{ formattedDate }}</span>
          </p>
        </div>
        <p class="balance__value">{{ formattedBalance }}</p>
      </div>

      <div class="movements">
        <div class="movements__row" v-for="(mov, i) in displayedMovements" :key="i">
          <div class="movements__type" :class="{'movements__type--deposit': mov.amount > 0, 'movements__type--withdrawal': mov.amount < 0}">
            {{ displayedMovements.length - i }} {{ mov.amount > 0 ? 'deposit' : 'withdrawal' }}
          </div>
          <div class="movements__date">{{ formatMovementDate(mov.date) }}</div>
          <div class="movements__value">{{ formatCurrency(mov.amount) }}</div>
        </div>
      </div>

      <div class="summary">
        <p class="summary__label">In</p>
        <p class="summary__value summary__value--in">{{ formattedIncome }}</p>
        <p class="summary__label">Out</p>
        <p class="summary__value summary__value--out">{{ formattedOutgoings }}</p>
        <p class="summary__label">Interest</p>
        <p class="summary__value summary__value--interest">{{ formattedInterest }}</p>
        <button class="btn--sort" @click="sortMovements">&downarrow; SORT</button>
      </div>

      <div class="operation operation--transfer">
        <h2>Transfer money</h2>
        <form @submit.prevent="transfer">
          <input type="text" v-model="transferTo" class="form__input form__input--to" />
          <input type="number" v-model="transferAmount" class="form__input form__input--amount" step="any" />
          <button class="form__btn form__btn--transfer">&rarr;</button>
          <label class="form__label">Transfer to</label>
          <label class="form__label">Amount</label>
        </form>
      </div>

      <div class="operation operation--loan">
        <h2>Request loan</h2>
        <form @submit.prevent="requestLoan">
          <input type="number" v-model="loanAmount" class="form__input form__input--loan-amount" step="any"/>
          <button class="form__btn form__btn--loan">&rarr;</button>
          <label class="form__label form__label--loan">Amount</label>
        </form>
      </div>

      <div class="operation operation--close">
        <h2>Close account</h2>
        <form @submit.prevent="closeAccount">
          <input type="text" v-model="closeUsername" class="form__input form__input--user" />
          <input type="password" v-model="closePin" maxlength="6" class="form__input form__input--pin" />
          <button class="form__btn form__btn--close">&rarr;</button>
          <label class="form__label">Confirm user</label>
          <label class="form__label">Confirm PIN</label>
        </form>
      </div>

      <p class="logout-timer">
        You will be logged out in <span class="timer">{{ logoutTimer }}</span>
      </p>
    </main>
  </div>
</template>
