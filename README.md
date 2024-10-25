# TrendsPy

Python library for accessing Google Trends data.

## Key Features

**Explore**
- Track popularity over time (`interest_over_time`)
- Analyze geographic distribution (`interest_by_region`)
- Compare interest across different timeframes and regions (multirange support)
- Get related queries and topics (`related_queries`, `related_topics`)

**Trending Now**
- Access current trending searches (`trending_now`, `trending_now_by_rss`)
- Get related news articles (`trending_now_news_by_ids`)
- Retrieve historical data for 500+ trending keywords with independent normalization (`trending_now_showcase_timeline`)

**Search Utilities**
- Find category IDs (`categories`)
- Search for location codes (`geo`)

**Flexible Time Formats**
- Custom intervals: `'now 123-H'`, `'today 45-d'`
- Date-based offsets: `'2024-02-01 10-d'`
- Standard ranges: `'2024-01-01 2024-12-31'`

## Installation

```bash
pip install trendspy
```

## Basic Usage

```python
from trendspy import Trends
tr = Trends()
df = tr.interest_over_time(['python', 'javascript'])
df.plot(title='Python vs JavaScript Interest Over Time', 
        figsize=(12, 6))
```

```python
# Analyze geographic distribution
geo_df = tr.interest_by_region('python')
```
```python
# Get related queries
related = tr.related_queries('python')
```

## Advanced Features

### Search Categories and Locations

```python
# Find technology-related categories
categories = tr.categories(find='technology')
# Output: [{'name': 'Computers & Electronics', 'id': '13'}, ...]

# Search for locations
locations = tr.geo(find='york')
# Output: [{'name': 'New York', 'id': 'US-NY'}, ...]

# Use in queries
df = tr.interest_over_time(
    'python',
    geo='US-NY',      # Found location ID
    cat='13'          # Found category ID
)
```

### Real-time Trending Searches and News

```python
# Get current trending searches in the US
trends = tr.trending_now(geo='US')

# Get trending searches with news articles
trends_with_news = tr.trending_now_by_rss(geo='US')
print(trends_with_news[0])  # First trending topic
print(trends_with_news[0].news[0])  # Associated news article

# Get news articles for specific trending topics
news = tr.trending_now_news_by_ids(
    trends[0].news_tokens,  # News tokens from trending topic
    max_news=3  # Number of articles to retrieve
)
for article in news:
    print(f"Title: {article.title}")
    print(f"Source: {article.source}")
    print(f"URL: {article.url}\n")
```

### Independent Historical Data for Multiple Keywords

```python
from trendspy import BatchPeriod

# Unlike standard interest_over_time where data is normalized across all keywords,
# trending_now_showcase_timeline provides independent data for each keyword
# (up to 500+ keywords in a single request)

keywords = ['keyword1', 'keyword2', ..., 'keyword500']

# Get independent historical data
df_24h = tr.trending_now_showcase_timeline(
    keywords,
    timeframe=BatchPeriod.Past24H  # 16-minute intervals
)

# Each keyword's data is normalized only to itself
df_24h.plot(
    subplots=True,
    layout=(5, 2),
    figsize=(15, 20),
    title="Independent Trend Lines"
)

# Available time windows:
# - Past4H:  ~30 points (8-minute intervals)
# - Past24H: ~90 points (16-minute intervals)
# - Past48H: ~180 points (16-minute intervals)
# - Past7D:  ~42 points (4-hour intervals)
```

### Geographic Analysis

```python
# Country-level data
country_df = tr.interest_by_region('python')

# State-level data for the US
state_df = tr.interest_by_region(
    'python',
    geo='US',
    resolution='REGION'
)

# City-level data for California
city_df = tr.interest_by_region(
    'python',
    geo='US-CA',
    resolution='CITY'
)
```

### Timeframe Formats

- Standard API timeframes: `'now 1-H'`, `'now 4-H'`, `'today 1-m'`, `'today 3-m'`, `'today 12-m'`
- Custom intervals:
  - Short-term (< 8 days): `'now 123-H'`, `'now 72-H'`
  - Long-term: `'today 45-d'`, `'today 90-d'`, `'today 18-m'`
  - Date-based: `'2024-02-01 10-d'`, `'2024-03-15 3-m'`
- Date ranges: `'2024-01-01 2024-12-31'`
- Hourly precision: `'2024-03-25T12 2024-03-25T15'` (for periods < 8 days)
- All available data: `'all'`

### Multirange Interest Over Time

Compare search interest across different time periods and regions:

```python
# Compare different time periods
timeframes = [
    '2024-01-25 12-d',    # 12-day period
    '2024-06-20 23-d'     # 23-day period
]
geo = ['US', 'GB']        # Compare US and UK

df = tr.interest_over_time(
    'python',
    timeframe=timeframes,
    geo=geo
)
```

Note: When using multiple timeframes, they must maintain consistent resolution and the maximum timeframe cannot be more than twice the length of the minimum timeframe.

### Proxy Support

TrendsPy supports the same proxy configuration as the `requests` library:

```python
# Initialize with proxy
tr = Trends(proxy="http://user:pass@10.10.1.10:3128")
# or
tr = Trends(proxy={
    "http": "http://10.10.1.10:3128",
    "https": "http://10.10.1.10:1080"
})

# Configure proxy after initialization
tr.set_proxy("http://10.10.1.10:3128")
```

## Documentation

For more examples and detailed API documentation, check out the Jupyter notebook in the repository: `basic_usage.ipynb`

## License

MIT License - see the [LICENSE](LICENSE) file for details.

## Disclaimer

This library is not affiliated with Google. Please ensure compliance with Google's terms of service when using this library.
