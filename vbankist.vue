<script>
export default {
  data() {
    return {
      accounts: [
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
    /**
     * Returns a welcome message based on the login status.
     * @returns {string} Welcome message.
     */
    welcomeMessage() {
      return this.isLoggedIn ? `Welcome back ${this.currentAccount.owner.split(" ")[0]}!` : 'Log in to get started';
    },
    
    /**
     * Returns the formatted balance of the current account.
     * @returns {string} Formatted balance.
     */
    formattedBalance() {
      return this.formatCurrency(this.calculateBalance());
    },

    /**
     * Returns the formatted total income of the current account.
     * @returns {string} Formatted income.
     */
    formattedIncome() {
      return this.formatCurrency(this.calculateSummary('incomes'));
    },

    /**
     * Returns the formatted total outgoings of the current account.
     * @returns {string} Formatted outgoings.
 */
    formattedOutgoings() {
      return this.formatCurrency(this.calculateSummary('outgoings'));
    },

    /**
     * Returns the formatted interest of the current account.
     * @returns {string} Formatted interest.
     */
    formattedInterest() {
      return this.formatCurrency(this.calculateInterest());
    },

    /**
     * Returns the current date in the format MM/DD/YYYY.
     * @returns {string} Formatted date.
     */
    formattedDate() {
      return new Date().toLocaleDateString();
    },

    /**
     * Returns the remaining logout timer duration in MM:SS format.
     * @returns {string} Logout timer duration.
     */
    logoutTimer() {
      const minutes = String(Math.floor(this.timerDuration / 60)).padStart(2, '0');
      const seconds = String(this.timerDuration % 60).padStart(2, '0');
      return `${minutes}:${seconds}`;
    },
  },
  methods: {
    /**
     * Creates usernames for all accounts.
     */
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

    /**
     * Handles the login process.
     */
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

    /**
     * Calculates the balance of the current account.
     * @returns {number} Balance.
     */
    calculateBalance() {
      return this.currentAccount ? this.currentAccount.movements.reduce((acc, mov) => acc + mov, 0) : 0;
    },

    /**
     * Calculates the total income or outgoings of the current account.
     * @param {string} type - 'incomes' or 'outgoings'.
     * @returns {number} Total income or outgoings.
     */
    calculateSummary(type) {
      if (!this.currentAccount) return 0;
      if (type === 'incomes') {
        return this.currentAccount.movements.filter(mov => mov > 0).reduce((acc, mov) => acc + mov, 0);
      } else if (type === 'outgoings') {
        return Math.abs(this.currentAccount.movements.filter(mov => mov < 0).reduce((acc, mov) => acc + mov, 0));
      }
    },

    /**
     * Calculates the interest of the current account.
     * @returns {number} Interest.
     */
    calculateInterest() {
      if (!this.currentAccount) return 0;
      return this.currentAccount.movements
        .filter(mov => mov > 0)
        .map(deposit => (deposit * this.currentAccount.interestRate) / 100)
        .filter(int => int >= 1)
        .reduce((acc, int) => acc + int, 0);
    },

    /**
     * Formats a value as currency.
     * @param {number} value - Value to format.
     * @returns {string} Formatted currency.
     */
    formatCurrency(value) {
      return new Intl.NumberFormat('en-US', { // Force 'en-US' locale
        style: 'currency',
        currency: this.currentAccount.currency,
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      }).format(value);
    },

    /**
     * Formats a date string to MM/DD/YYYY.
     * @param {string} date - Date string to format.
     * @returns {string} Formatted date.
     */
    formatMovementDate(date) {
      const targetDate = new Date(date);
      // Format the date to MM/DD/YYYY
      const month = String(targetDate.getMonth() + 1).padStart(2, '0'); // Months are zero-indexed
      const day = String(targetDate.getDate()).padStart(2, '0');
      const year = targetDate.getFullYear();

      return `${month}/${day}/${year}`;
    },

    /**
     * Starts the logout timer.
     */
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

    /**
     * Handles the transfer process.
     */
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

    /**
     * Handles the loan request process.
     */
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

    /**
     * Handles the account closure process.
     */
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

    /**
     * Sorts the displayed movements by amount or date.
     */
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

    /**
     * Updates the UI with the current account movements.
     */
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