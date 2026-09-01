# Stock Exchange

## Requirements

| Functional      | Non functional      |
| ------------- | ------------- |
| Placing a new limit order in the normal trading hours  | Availability 99.99% |
| Canceling an order | Fault tolerance |
| User wallet to make sure user has enough funds. | Latency 99th percentile |
|  | Security |
|  |  |

### Assumsions

- 10K traders at the same time.
- support 100 symbols.
- billions of orders per day.
- Max 1M shares of Apple stock in 1 day.

### Calculations

- No of hours per working day 9:30am ~ 4pm = 6.5hrs
- No of seconds // // // = 6.5 * 3600 = 23,400 seconds
- QPS = orders / time = 1 billion / 23,400 = 43K
- Peak QPS = 5 * QPS = 215K
