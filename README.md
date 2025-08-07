Football Transfer Market Analysis: Market Value vs. Actual Fees
===============================================================

Project Objective
-----------------

The primary objective of this project is to investigate the accuracy of Transfermarkt's estimated player market values compared to the actual transfer fees paid in professional football. We aim to identify the discrepancies between these two metrics and explore the factors that contribute to these differences, with a particular focus on understanding the underlying data methodologies.

Data Source
-----------

The analysis utilizes the "David Cariboo / Player Scores" dataset from Kaggle, comprising several CSV files:

-   `players.csv`: Player biographical information.

-   `player_valuations.csv`: Historical market value estimates for players.

-   `transfers.csv`: Records of player transfers, including actual fees and Transfermarkt's market value at the time of transfer.

-   `appearances.csv`: Player performance statistics per game.

-   `clubs.csv`: Club information.

Analysis Progress & Key Findings
--------------------------------

### 1\. Data Loading and Initial Filtering

-   All relevant CSV files (`players`, `player_valuations`, `transfers`, `appearances`, `clubs`) were loaded into pandas DataFrames.

-   Date columns (`date_of_birth`, `date` in `player_valuations`, `transfer_date` in `transfers`) were converted to `datetime` objects for accurate time-series analysis.

-   The `transfers.csv` DataFrame was filtered to `df_transfers_paid`, including only transfers where `transfer_fee` was `notna()` and greater than `0`, and `market_value_in_eur` was also `notna()`. This resulted in **8,872 paid transfer records** for analysis.

### 2\. Calculating the Net Difference

-   A new column, `net_difference`, was created in `df_transfers_paid`, defined as: `net_difference = transfer_fee_in_eur - market_value_in_eur` (Actual Transfer Fee - Transfermarkt Market Value at time of transfer).

-   The **mean `net_difference` was found to be approximately €696,302**. This positive mean suggests that, on average, actual transfer fees tend to be higher than Transfermarkt's estimates.

### 3\. Initial Distribution Analysis of `net_difference`

-   A histogram of `net_difference` was plotted, with the x-axis range adjusted to focus on the central distribution (e.g., `(-37M, 37M)`).

-   **Observation:** The distribution was found to be "very clumped" around zero, with the vast majority of records falling between -€1M and €1M.

-   The **median `net_difference` was found to be €0**. This is a critical finding, indicating that for half of all paid transfers, the actual fee was equal to or less than the market value.

### 4\. Quantifying Exact Matches and Clumpiness

-   **Exact Matches:** Out of 8,872 paid transfers, **581 records (approximately 6.55%)** showed a `net_difference` of exactly `0` (i.e., `transfer_fee == market_value_in_eur`).

-   **Clumpiness:** Further inspection of the histogram with more bins revealed distinct "clumps" of data points at specific, rounded `net_difference` values (e.g., ±€500K, ±€1M, and smaller concentrations around ±€100K).

-   **Implication:** These findings strongly suggest that Transfermarkt, the data source, likely employs **data imputation or rounding strategies** when exact transfer fees are unknown or to align their market values with common negotiation increments. This means the `market_value_in_eur` in `transfers.csv` is not always a purely independent estimate, but sometimes an adjusted or imputed value.

### 5\. Data Merging and Mismatch Resolution

-   An attempt to merge `df_transfers_paid` with `df_players` (to bring in player attributes) initially resulted in only 759 matched records, which was a significant concern given the 8,872 transfers.

-   Through debugging, it was confirmed that `player_id` data types were consistent (`int64`).

-   After further checks, the merge issue resolved itself, and the merge was successfully completed. The `df_merged` DataFrame now contains **8,872 records**, with player-specific attributes (like `player_name`, `date_of_birth`, `sub_position`) largely populated, indicating a successful and comprehensive merge.

### 6\. Analysis by Player Attributes (Age & Position)

-   `age_at_transfer` was calculated from `transfer_date` and `date_of_birth`.

-   Initial scatter plots and box plots of `net_difference` against `age_at_transfer` and `position` (using the existing `position` column) were performed.

-   **Observation:** For the overall dataset, the **median `net_difference` remained close to €0** across most age groups and positions.

-   **Implication:** This suggests that broad player attributes like age and general position do not, for the typical transfer, explain why a fee would be significantly above or below market value. The "clumpiness" and the overall median of zero persist across these categories, implying that Transfermarkt's estimates are often close, or imputed to be close. However, the presence of outliers remains a key area of interest.

### 7\. Analysis by Nationality

-   The top N (e.g., 10-20) most frequent nationalities were identified from `df_merged`.

-   The mean and median `net_difference` were calculated for transfers involving players from these top nationalities.

-   **Key Finding: England stands out significantly.** The **median `net_difference` for English players was found to be approximately €1.5 million**, which is substantially higher than the overall median of €0 and also significantly higher than other top footballing nations (e.g., Brazil's median of €425K).

-   **Implication:** This provides strong evidence that English players, on average and typically, command a higher price than their Transfermarkt valuation. This aligns with real-world understanding of the Premier League's financial power and demand for homegrown talent.

-   **Outlier Impact (Revisited):** The analysis also showed that for many other countries, while the median `net_difference` might be close to zero, the mean is often pulled significantly higher by a few extreme positive outliers.

### 8\. Outlier Removal Experiment (Top & Bottom 5% per Country)

-   To further understand the impact of extreme values, the top 5% and bottom 5% of `net_difference` values were removed *for each country group* using `groupby().transform()`.

-   **Observation:** Surprisingly, removing these extreme tails **did not change the mean `net_difference` as much as initially expected** for many countries.

-   **Implication:** This suggests that the influence on the mean might not solely come from a few isolated, super-extreme outliers, but also from a more dispersed set of larger values in the tails, or from the inherent "clumpiness" of the data at rounded figures, which are not necessarily removed by a 5% trim.

Next Steps (Future Work)
------------------------

The project has successfully identified key characteristics of the `net_difference` and highlighted England as a significant outlier. Future steps could include:

-   **Deeper Outlier Characterization:** A more focused analysis of the specific transfers (players, clubs, seasons) that fall into the very highest positive `net_difference` categories, especially for England and other countries.

-   **Incorporating Performance Data:** Merging `df_appearances` to bring in per-season player performance metrics (goals, assists, minutes played, etc.) to see if these factors explain more of the `net_difference`.

-   **Modeling:** Building a regression model to predict `transfer_fee` using `market_value_in_eur` and other player attributes, and then analyzing the residuals to identify unexplained variance.

-   **External Factors Research:** Further research into real-world factors (e.g., Premier League broadcasting deals, homegrown player rules, club financial situations, agent influence) to provide qualitative explanations for the observed quantitative patterns.
