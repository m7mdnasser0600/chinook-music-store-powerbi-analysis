# 🎸 Chinook Music Store – Sales Analysis Dashboard (Power BI)

An end-to-end business intelligence project that turns the raw transactional data of the **Chinook digital music store** into an interactive, multi-page Power BI dashboard covering sales performance, artist and catalog analysis, customer behavior, and revenue trends over time.

**Pipeline:** SQL Server (data source) → Power Query (cleaning and shaping) → Data Model (fact + dimensions) → DAX (measures) → Power BI Dashboard (designed in Figma)

<!-- Add your best dashboard screenshot here -->
![Executive Overview](images/01_executive_overview.png)

---

## 📑 Table of Contents
1. [Project Objective](#-project-objective)
2. [Key Insights](#-key-insights)
3. [Tools Used](#-tools-used)
4. [Repository Structure](#-repository-structure)
5. [Dataset Description](#-dataset-description)
6. [Data Preparation (SQL and Power Query)](#-data-preparation-sql-and-power-query)
7. [Data Modeling](#-data-modeling)
8. [DAX Measures](#-dax-measures)
9. [Dashboard Pages](#-dashboard-pages)
10. [Interactive Features](#-interactive-features)
11. [How to Run This Project](#-how-to-run-this-project)
12. [Documentation and Presentation](#-documentation-and-presentation)
13. [Author](#-author)
14. [Acknowledgements](#-acknowledgements)

---

## 🎯 Project Objective

Analyze the sales data of the Chinook music store to understand:

- **Sales performance:** total revenue, orders, and average order value
- **Music catalog characteristics:** artists, albums, genres, media types, and track lengths
- **Customer behavior:** who the top customers are, where they are located, and how support representatives perform
- **Time-based trends:** revenue growth, year-over-year change, and seasonality

---

## 💡 Key Insights

| Area | Finding |
|---|---|
| Revenue trend | Revenue is stable over time, with no strong growth trend |
| Top artists | **Iron Maiden** leads, followed by U2, Metallica, and Led Zeppelin |
| Top genres | **Rock** leads, followed by Latin, Metal, and Alternative & Punk |
| Concentration | Revenue is highly concentrated in a few artists and genres |
| Media type | MPEG audio is both the most common and the best-selling format |
| Track length | Medium tracks (3–7 min) are the most common and generate the highest sales |
| Geography | **USA** is the strongest market, followed by Canada, Brazil, and France |
| Seasonality | Sales peak mid-month and mid-year |

---

## 🛠 Tools Used

| Tool | Purpose |
|---|---|
| **SQL (SQL Server / T-SQL)** | Create and populate the Chinook database |
| **Power BI Desktop** | Data modeling, visualization, and dashboard creation |
| **Power Query** | Data cleaning, transformation, and table merging |
| **DAX** | Calculated columns, date table, and measures |
| **Figma** | Dashboard layout and visual style design |

---

## 📂 Repository Structure

```
chinook-music-store-powerbi-analysis/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── data/
│   └── chinook_database.sql             # T-SQL script: database, tables, keys, data
│
├── powerbi/
│   └── Chinook_Sales_Dashboard.pbix     # Final Power BI report
│
├── powerquery/
│   ├── invoiceline_Facttable.pq         # Fact table query (Revenue column)
│   └── Dim_Track.pq                     # Track, Album, Artist, Genre, MediaType merge
│
├── dax/
│   └── measures.dax                     # All measures, calculated columns, date table
│
├── docs/
│   ├── Chinook_Documentation.pdf        # Full project documentation
│   ├── Chinook_Presentation.pptx        # Presentation of findings
│   └── Chinook_Presentation.pdf         # PDF version (previews directly on GitHub)
│
└── images/
    ├── 01_executive_overview.png
    ├── 02_artist_analysis.png
    ├── 03_music_catalog.png
    ├── 04_customer_analysis.png
    ├── 05_time_analysis.png
    └── data_model.png
```

---

## 🗃 Dataset Description

The dataset represents the sales operations of the **Chinook** digital music store. Each **invoice** is a purchase by a customer, and each **invoice line** is one track purchased within that invoice.

The source script (`data/chinook_database.sql`) is written in T-SQL. It creates the `Chinook` database, 11 tables, primary and foreign keys, indexes, and inserts all the data.

| Table | Description |
|---|---|
| `Artist` | Music artists |
| `Album` | Albums, each belonging to one artist |
| `Track` | Individual tracks with genre, media type, composer, duration (ms), size (bytes), and unit price |
| `Genre` | Genre categories used to classify tracks |
| `MediaType` | Track format (for example, MPEG audio file) |
| `Customer` | Customer name, location, contact details, and assigned support representative |
| `Employee` | Employees, including customer support representatives |
| `Invoice` | Order-level data: invoice date, customer, billing address, and total |
| `InvoiceLine` | Items purchased per invoice: track, unit price, and quantity |
| `Playlist`, `PlaylistTrack` | Playlists (not required for this analysis, so removed from the model) |

---

## 🧹 Data Preparation (SQL and Power Query)

### 1. SQL – creating the source
Run `data/chinook_database.sql` on SQL Server. It creates the database, tables, relationships, and sample data that Power BI connects to.

### 2. Power Query – cleaning and shaping
All transformations were done in Power Query. The M code of each query is saved in the [`powerquery/`](powerquery) folder.

Steps applied:

- Adjusted column data types to ensure correct calculations
- Renamed tables and columns for readability
- Removed unnecessary tables and columns (such as `Playlist` and `PlaylistTrack`)
- Created additional columns required for analysis

#### Revenue column (`invoiceline_Facttable`)
A `Revenue` column was added to the fact table so revenue can be summed across any dimension (artist, genre, customer, country):

```m
Table.AddColumn(Source, "Revenue", each [UnitPrice] * [Quantity], Currency.Type)
```

#### Dim_Track dimension (denormalization)
The normalized tables **Track, Album, Artist, Genre, and MediaType** were merged into one `Dim_Track` table. This puts every track-related attribute in a single table and reduces the number of relationships in the model.

| Merged from | Join key | Column(s) brought in |
|---|---|---|
| `Track` | – (base table) | Track name, Composer, Milliseconds, Bytes, UnitPrice |
| `Album` | `AlbumId` | Album title, `ArtistId` |
| `Artist` | `ArtistId` | Artist name |
| `Genre` | `GenreId` | Genre name |
| `MediaType` | `MediaTypeId` | Media type name |

Resulting columns include: Track, Album, Artist, Genre, MediaType, Composer, Milliseconds, Bytes, and Track Length.

#### Track_Length column (`Dim_Track`, DAX calculated column)
Tracks are categorized by duration:

| Category | Rule |
|---|---|
| Short | shorter than 3 minutes |
| Medium | 3 to 7 minutes |
| Long | longer than 7 minutes |

```dax
Track_Length =
SWITCH(
    TRUE(),
    Dim_Track[Milliseconds] < 180000, "Short",
    Dim_Track[Milliseconds] >= 180000 && Dim_Track[Milliseconds] < 420000, "Medium",
    Dim_Track[Milliseconds] >= 420000, "Long"
)
```

#### Date dimension (`Dim_Date`)
Created with DAX to enable time-intelligence calculations (YTD, QTD, MTD, YoY):

```dax
Dim_Date = CALENDARAUTO()
```

---

## 🧩 Data Modeling

The model is built around **`invoiceline_Facttable`** as the central fact table, since it holds the lowest level of sales transactions (one row per purchased track).

![Data Model](images/data_model.png)

### Tables

| Role | Table | Description |
|---|---|---|
| **Fact** | `invoiceline_Facttable` | Sales transactions: InvoiceId, InvoiceLineId, TrackId, Quantity, UnitPrice, Revenue |
| Dimension | `Dim_Track` | Track, album, artist, genre, media type, duration, track length category |
| Dimension | `Dim_Customer` | Customer name, location, and contact details |
| Dimension | `Dim_Date` | Generated calendar table (Date, Day, Month, Month Format, Quarter, Year) |
| Dimension | `employee` | Employees acting as customer support representatives |
| Bridge | `invoice` | Order-level data (invoice date, billing location, customer ID) linking sales lines to customers and dates |

### Relationships

| From (many side) | To (one side) | Key | Cardinality |
|---|---|---|---|
| `invoiceline_Facttable` | `Dim_Track` | `TrackId` | Many-to-one |
| `invoiceline_Facttable` | `invoice` | `InvoiceId` | Many-to-one |
| `invoice` | `Dim_Customer` | `CustomerId` | Many-to-one |
| `invoice` | `Dim_Date` | Invoice date → `Date` | Many-to-one |
| `Dim_Customer` | `employee` | `SupportRepId` → `EmployeeId` | Many-to-one |

Because `invoice` sits between the fact table and the customer/date tables, the model is a small **snowflake** extension of a star schema. These relationships let Power BI filter and aggregate sales by artist, genre, customer, country, support representative, and time.

---

## 📐 DAX Measures

All measures are also available in [`dax/measures.dax`](dax/measures.dax).

### Core measures
```dax
Total Revenue = SUM(invoiceline_Facttable[Revenue])
Total Orders = DISTINCTCOUNT(invoice[InvoiceId])
Number of Customers = DISTINCTCOUNT(Dim_Customer[CustomerId])
Average Order Value = DIVIDE([Total Revenue], [Total Orders])
Avg Revenue per Customer = DIVIDE([Total Revenue], [Number of Customers])
Orders per Customer = DIVIDE([Total Orders], [Number of Customers])
```

### Catalog and quantity measures
```dax
Total Quantity = SUM(invoiceline_Facttable[Quantity])
Album Count = DISTINCTCOUNT(Dim_Track[Album])
Track Count = COUNT(Dim_Track[TrackId])
Avg Track Duration = AVERAGE(Dim_Track[Milliseconds]) / 1000 / 60   -- minutes
Avg Track Revenue = DIVIDE([Total Revenue], [Total Quantity])
```

### Time intelligence
```dax
YTD = TOTALYTD([Total Revenue], Dim_Date[Date])
QTD = TOTALQTD([Total Revenue], Dim_Date[Date])
MTD = TOTALMTD([Total Revenue], Dim_Date[Date])
YoY = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(Dim_Date[Date]))   -- revenue of the same period last year
Growth % = DIVIDE([Total Revenue] - [YoY], [YoY])
```

### Dynamic field parameters
Field parameters let users switch the metric shown in a visual:

- Artist catalog size: album count / track count / average track duration
- Top 10 Artists: revenue / quantity sold
- Media Type chart: track count / revenue
- Genre comparison: track count / album count / average track duration
- Support representative performance: revenue / quantity sold / total orders

---

## 📊 Dashboard Pages

The dashboard follows a top-down approach: high-level KPIs first, then artist, catalog, customer, and time detail. It has **5 analytical pages**, each available in **dark and light mode**.

### 1. Executive Overview
- KPI cards: Total Revenue, Total Orders, Total Customers, Average Order Value
- Top 10 Genres by Revenue
- Top 10 Artists by Revenue
- Revenue by Country (map)
- Slicers: Year, Artist, Genre, Country

![Executive Overview](images/01_executive_overview.png)

### 2. Artist Analysis
- KPI cards: Total Revenue, Album Count, Tracks Sold, Average Revenue per Track
- Artists with the largest catalog (dynamic parameter)
- Top 10 Artists (switch between revenue and quantity)
- Top 10 Albums by Revenue
- Table of top-selling tracks (track, artist, album, revenue)
- Slicers: Artist, Genre

![Artist Analysis](images/02_artist_analysis.png)

### 3. Music Catalog
- KPI cards: Track Count, Genre Count, Average Track Duration, Artist Count
- Media Type distribution (switch between track count and revenue)
- Track Length category chart
- Genre revenue treemap
- Genre comparison (dynamic parameter)
- Slicers: Genre, Artist

![Music Catalog](images/03_music_catalog.png)

### 4. Customer Analysis
- KPI cards: Total Customers, Total Revenue, Average Customer Value, Average Order Value
- Support representative performance (dynamic parameter)
- Top 10 Customers by Revenue
- Number of customers by country (map)
- Slicers: Genre, Country, Customer Name

![Customer Analysis](images/04_customer_analysis.png)

### 5. Time Analysis
- KPI cards: Total Revenue, YoY Revenue, YTD Revenue, Growth %
- Revenue trend line chart across the date hierarchy
- Growth % vs. YoY revenue over time
- Matrix: revenue by year, quarter, month, and day

![Time Analysis](images/05_time_analysis.png)

---

## ⚙️ Interactive Features

- Slicers for year, genre, artist, country, and customer
- Dynamic field parameters to switch between metrics
- Cross-filtering between visuals
- Hierarchical drill-down in time-based charts
- Navigation buttons between pages and a dark/light mode switch

---

## ▶️ How to Run This Project

**Just view the dashboard**
1. Download `powerbi/Chinook_Sales_Dashboard.pbix`.
2. Open it with [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free, Windows). The data is already imported into the file.

**Rebuild from the source data**
1. Open SQL Server (or SSMS / Azure Data Studio) and run `data/chinook_database.sql`.
2. In Power BI Desktop, choose **Get Data → SQL Server** and connect to the `Chinook` database.
3. Recreate the queries using the code in `powerquery/` (Home → Transform data → Advanced Editor).
4. Add the DAX from `dax/measures.dax`.
5. Rebuild the relationships shown in [Data Modeling](#-data-modeling).

---

## 📚 Documentation and Presentation

- 📄 [Full project documentation (PDF)](docs/Chinook_Documentation.pdf)
- 🎤 [Presentation (PDF)](docs/Chinook_Presentation.pdf) · [PowerPoint file](docs/Chinook_Presentation.pptx)

---

## 👤 Author

**Mohamed AbdEl-Nasser**
- LinkedIn: [your-profile-link]
- GitHub: [your-github-link]

---

## 🙏 Acknowledgements

The Chinook sample database was created by Luis Rocha and is available at [lerocha/chinook-database](https://github.com/lerocha/chinook-database).
