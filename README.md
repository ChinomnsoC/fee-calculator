# fee-calculator

We want to bill customers for all days users were active in the month (including any activation and deactivation dates, since the user had some access on those days).
• We do need to support historical calculations (previous dates)
• We only charge whole cents
Notes
Here's an idea of how we might go about this:
1. Calculate a daily rate for the subscription tier
2. For each day of the month, identify which users had an active subscription on that day
3. Multiply the number of active users for the day by the daily rate to calculate the total for the day
4. Return the running total for the month at the end

export interface User {
  id: number;
  name: string;
  activatedOn: Date;
  deactivatedOn: Date | null;
  customerId: number;
}

export interface Subscription {
  id: number;
  customerId: number;
  monthlyPriceInCents: number;
}

/**
 * Computes the monthly charge for a given subscription.
 *
 * @returns The total monthly bill for the customer in cents, rounded
 * to the nearest cent. For example, a bill of $20.00 should return 2000.
 * If there are no active users or the subscription is null, returns 0.
 *
 * @param month - Always present
 *   Has the following structure:
 *   "2022-04"  // April 2022 in YYYY-MM format
 *
 * @param subscription - May be null
 *   If present, has the following structure (see Subscription interface):
 *   {
 *     'id': 763,
 *     'customerId': 328,
 *     'monthlyPriceInCents': 359  // price per active user per month
 *   }
 *
 * @param users - May be empty, but not null
 *   Has the following structure (see User interface):
 *   [
 *     {
 *       id: 1,
 *       name: "Employee #1",
 *       customerId: 1,
 *   
 *       // when this user started
 *       activatedOn: new Date("2021-11-04"),
 *   
 *       // last day to bill for user
 *       // should bill up to and including this date
 *       // since user had some access on this date
 *       deactivatedOn: new Date("2022-04-10")
 *     },
 *     {
 *       id: 2,
 *       name: "Employee #2",
 *       customerId: 1,
 *   
 *       // when this user started
 *       activatedOn: new Date("2021-12-04"),
 *   
 *       // hasn't been deactivated yet
 *       deactivatedOn: null
 *     },
 *   ]
 */
export function monthlyCharge(yearMonth: string, subscription: Subscription | null, users: User[]): number {
  // your code here!
  if (!yearMonth) {
    throw new Error('Missing required arguments')
  }
//   extract date from yearMonth
  const date = new Date(yearMonth.includes("-") ? yearMonth: `${yearMonth} 01`);
  const month = date.getMonth();
  const year = date.getFullYear();
  
  const dailyFee = subscription? subscription.monthlyPriceInCents/getDaysInMonth(month, year) : 0;
  let totalCharge = 0
  
  for (let day = 1; day <= getDaysInMonth(month, year); day++){
    const currentDate = new Date(year, month -1, day);
    const activeUsers = users.filter(user => isActiveOn(user, currentDate));
    totalCharge += Math.round(activeUsers.length * dailyFee);
  }
  return totalCharge
}

function isActiveOn(user:User, date:Date ): boolean {
  const userActiveDate = new Date(user.activatedOn)
  const userDeactiveDate = user.deactivatedOn? new Date(user.deactivatedOn): null;
  return userActiveDate <= date && (!userDeactiveDate || userDeactiveDate >= date)
  
}
/*******************
* Helper functions *
*******************/
function getDaysInMonth(month: number, year: number){
  return new Date(year, month, 0).getDate();
}
/**
  Takes a Date instance and returns a Date which is the first day
  of that month. For example:

  firstDayOfMonth(new Date(2022, 3, 17)) // => new Date(2022, 3, 1)

  Input type: Date
  Output type: Date
**/
function firstDayOfMonth(date: Date): Date {
  return new Date(date.getFullYear(), date.getMonth(), 1)
}

/**
  Takes a Date object and returns a Date which is the last day
  of that month. For example:

  lastDayOfMonth(new Date(2022, 3, 17)) // => new Date(2022, 3, 31)

  Input type: Date
  Output type: Date
**/
function lastDayOfMonth(date: Date): Date {
  return new Date(date.getFullYear(), date.getMonth() + 1, 0)
}

/**
  Takes a Date object and returns a Date which is the next day.
  For example:

  nextDay(new Date(2022, 3, 17)) // => new Date(2022, 3, 18)
  nextDay(new Date(2022, 3, 31)) // => new Date(2022, 4, 1)

  Input type: Date
  Output type: Date
**/
function nextDay(date: Date): Date {
  return new Date(date.getFullYear(), date.getMonth(), date.getDate() + 1)
}
