# Stock Data Downloader and Technical Analysis

This project is a Python-based tool designed to download historical stock and financial instrument data from Yahoo Finance, process it, and compute technical indicators for analysis. It supports multiple time intervals, prioritizes downloads based on data freshness, and includes robust error handling and data validation. The processed data is filtered and enriched with technical indicators, making it suitable for financial analysis, algorithmic trading, or machine learning applications.

## Features

- **Data Download**: Retrieves historical data for a wide range of financial instruments, including stocks, ETFs, currencies, futures, and indices, using the `yfinance` library.
- **Multiple Time Intervals**: Supports data intervals of 1 minute, 5 minutes, 15 minutes, 30 minutes, 1 hour, and 1 day.
- **Priority-Based Downloading**: Prioritizes tickers based on the age of existing data, ensuring fresher datasets are updated first.
- **Rate Limiting**: Implements a smart rate limiter to prevent API throttling or bans, with dynamic delays based on errors.
- **Data Validation and Filtering**: Uses the Interquartile Range (IQR) method to filter out datasets with insufficient data and copies valid datasets to a separate directory.
- **Technical Indicators**: Computes a variety of technical indicators, including RSI, Stochastic Oscillator, Mean Reversion Channel, SMA, EMA, volatility, momentum, and Heston model parameters.
- **Visualization**: Generates plots to visualize data coverage and distribution of dataset lengths.
- **Error Handling**: Tracks errors per ticker, implements cooldown periods for problematic tickers, and logs all operations for debugging.
- **Parallel Processing**: Uses `joblib` for parallel computation of technical indicators to improve performance.
- **Resume Capability**: Supports resuming interrupted downloads by tracking progress in a state file.

## Project Structure

- **`datasets/`**: Stores raw downloaded data, organized by time interval (e.g., `1m`, `1d`).
- **`datasets_enough_data/`**: Contains filtered datasets with sufficient data based on IQR thresholds.
- **`datasets_technicals/`**: Stores datasets enriched with technical indicators.
- **`datasets_edit/`**: Directory for modified or custom datasets (not used in the provided code).
- **`tickers_metadata.json`**: Tracks metadata for each ticker, including last update times and error timestamps.
- **`downloader.log`**: Logs all operations, including errors and progress.
- **`download_state.json`**: Tracks download progress for resuming interrupted sessions.
- **`RISK_FREE_TICKER.csv`**: Stores historical data for the risk-free rate (13-week Treasury Bill, `^IRX`).

## Dependencies

The project relies on the following Python libraries:
- `yfinance`: For downloading financial data from Yahoo Finance.
- `pandas`: For data manipulation and storage.
- `numpy`: For numerical computations.
- `matplotlib`: For plotting data coverage and analysis results.
- `joblib`: For parallel processing of technical indicators.
- `tqdm`: For progress bars during data processing.
- `requests`: For optimized HTTP sessions.
- `concurrent.futures`: For parallel download execution.
- `logging`: For operation logging.
- `shutil`: For file copying during data filtering.

Install dependencies using:
```bash
pip install yfinance pandas numpy matplotlib joblib tqdm requests
```

## Usage

1. **Configure Tickers**:
   - The script defines lists of tickers for various categories: `PARIS_TICKERS` (French stocks), `US_ETF` (U.S. ETFs), `EU_ETF` (European ETFs), `CURRENCY_AND_FUTURES` (currencies and futures), and `US_TICKERS` (U.S. stocks).
   - All tickers are combined into a master `TICKERS` list, excluding blacklisted tickers (e.g., `AXA SA`).
   - The risk-free rate ticker (`^IRX`) is downloaded separately and saved as `RISK_FREE_TICKER.csv`.

2. **Download Data**:
   - The `PriorityDownloader` class manages data downloads with several execution modes:
     - `run_with_resume()`: Sequential download with resume capability (default).
     - `run_cross_interval(max_batches_per_run)`: Alternates between intervals to ensure balanced updates.
     - `run_with_quotas(total_batches)`: Allocates quotas per interval based on ticker counts.
     - `run_parallel(max_workers)`: Downloads data in parallel across intervals (use cautiously due to API limits).
   - To run the default mode:
     ```python
     downloader = PriorityDownloader(batch_size=15)
     downloader.run_with_resume()
     ```

3. **Filter Data**:
   - The `analyze_and_plot_csv_files_by_iqr` function analyzes dataset lengths, computes IQR-based thresholds, and plots the distribution of dataset sizes.
   - The `copy_valid_csv_files` function copies datasets with sufficient data (above the threshold) to `datasets_enough_data/`.
   - Run the filtering step:
     ```python
     analyze_and_plot_csv_files_by_iqr(DATA_DIR, INTERVALS, iqr_factor=1.5, top_n=10)
     copy_valid_csv_files(DATA_DIR, INTERVALS, seuils, DATA_DIR_ENOUGH_DATA)
     ```

4. **Compute Technical Indicators**:
   - The `process_all_data` function loads filtered datasets, computes technical indicators, and saves the results to `datasets_technicals/`.
   - Indicators include:
     - RSI (14-period)
     - Stochastic Oscillator (%K and %D, 14-period)
     - Mean Reversion Channel (20-period)
     - Simple Moving Averages (10, 20, 50, 100, 200 periods)
     - Exponential Moving Averages (10, 20, 50 periods)
     - Volatility (20, 50, 100, 360 periods)
     - Momentum (20, 50, 100, 252, 360 periods)
     - Heston model parameters (theta, sigma_v, kappa)
   - Run the indicator computation:
     ```python
     process_all_data()
     ```

5. **Visualize Coverage**:
   - The `plot_simple_coverage` function generates a plot showing the temporal coverage of each ticker, sorted by the number of data points.
   - Run the visualization for a specific interval (e.g., `1d`):
     ```python
     fig = plot_simple_coverage(TECH_DATA_DIR, DATA_INTERVAL)
     plt.show()
     ```

## Configuration

- **Intervals**: Defined in `INTERVALS` (`['1m', '5m', '15m', '30m', '1h', '1d']`).
- **Update Thresholds**: Specified in `UPDATE_THRESHOLDS` to determine when data is considered outdated.
- **Batch Sizes**: Customizable per interval in `PriorityDownloader.batch_sizes` to balance API load.
- **Error Handling**:
  - `MAX_CONSECUTIVE_ERRORS`: Limits retries after 5 consecutive failures.
  - `ERROR_COOLDOWN`: Enforces a 2-hour cooldown for problematic tickers.
  - `MIN_NB_LINES`: Ensures datasets have at least 200 rows (used in filtering).
- **User Agents**: Rotates through a list of `USER_AGENTS` to avoid API blocking.
- **IQR Factor**: Set to 1.5 for filtering datasets, adjustable in `analyze_and_plot_csv_files_by_iqr`.

## Logging

All operations are logged to `downloader.log` with timestamps, log levels, and detailed messages. This includes:
- Download progress and errors.
- Data processing and filtering steps.
- Indicator computation and file saving.

## Limitations

- **API Limits**: Yahoo Finance may impose rate limits, mitigated by the `SmartRateLimiter` but still a potential bottleneck.
- **Data Gaps**: Some tickers may have incomplete or missing data, handled by the error tracking system.
- **Parallel Execution**: The `run_parallel` mode may overwhelm the API, so use it sparingly.
- **Memory Usage**: Processing large datasets with many tickers and intervals can be memory-intensive.
- **Heston Parameters**: May produce NaN values for datasets with insufficient volatility data.

## Contributing

Contributions are welcome! To contribute:
1. Fork the repository.
2. Create a feature branch (`git checkout -b feature/your-feature`).
3. Commit your changes (`git commit -m "Add your feature"`).
4. Push to the branch (`git push origin feature/your-feature`).
5. Open a pull request.

Please include tests and update documentation as needed.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## Contact

For questions or issues, please open an issue on GitHub or contact the maintainer at [your-email@example.com].