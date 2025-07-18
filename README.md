# User Journey Cohort Analysis (Web Session Data)

---

## **Introduction**

At least in this case, it's not so much about the destination... it's about how one gets there.

A **user journey** illustrates the sequence of steps a user takes while interacting with a website. Understanding *how users interact* over many sessions is key to discovering consistent behaviors that ultimately lead to **purchases**.

This project analyzes detailed web session data from a SaaS (Software as a Service) product. *SaaS* refers to applications delivered via the internet, such as Salesforce, Microsoft Office 365, or Dropbox, where users access software through their browser as opposed to a program installation.

By mapping user journeys across *hundreds of thousands* of sessions, this study uncovers **cohort-level** behavioral patterns and reveals what truly drives users to make purchases.

---

## **About the Dataset**

The data for this study was exported from **Google Analytics (BigQuery)**, a platform that enables website owners the ability to store very large volumes of web activity using Google's cloud database infrastructure.

This dataset contains detailed website session data for more than 800,000 distinct users, capturing their activity over the course of an entire year.

To optimize performance in Python, only the most relevant columns were extracted. Importing the entirety of the extremely large BigQuery dataset would be impractical (and often impossible). This condensed structure provides a user-level and session-level analysis of how website visitors navigated the website and what influenced their likelihood to purchase.

---

## **Project Goals**

- **Segment users into behavioral cohorts** based on traffic medium (i.e., how they accessed the website).
- **Map user journeys** step by step, tracking each page viewed from the landing page to a purchase, or from the landing page to an exit (without a purchase).
- **Quantify drop-off rates** for each cohort at every step in the journey.
- **Identify which cohorts are most likely to make a purchase** and highlight differences.

---

## **Table of Contents**

- [Introduction](#introduction)
- [About the Dataset](#about-the-dataset)
- [Project Goals](#project-goals)
- [Tools Used](#tools-used)
- [Project Files](#project-files)
- [Data Loading, Preparation, and Cleaning](#data-loading-preparation-and-cleaning)
- [Step 1: Purchase Rate by Path](#step-1-purchase-rate-by-path)
- [Step 2: Purchase Rate by Path Length](#step-2-purchase-rate-by-path-length)
- [Step 3: Drop-Off Percentage at Each Page Viewed](#step-3-drop-off-percentage-at-each-page-viewed)
- [Additional Columns Import from BigQuery](#additional-columns-import-from-bigquery)
- [Step 4: Purchase Rate by Source, Medium, Source + Medium, and Entry Page](#step-4-purchase-rate-by-source-medium-source--medium-and-entry-page)
- [Step 5: Assign Cohorts by Traffic Medium](#step-5-assign-cohorts-by-traffic-medium)
- [Step 6: Drop-Off Percentage at Each Page Viewed](#step-6-drop-off-percentage-at-each-page-viewed)
- [Tableau Dashboard](#tableau-dashboard)
- [Final Thoughts](#final-thoughts)
- [Tableau Dashboard Link](#tableau-dashboard-link)

---

## **Tools Used**

- **BigQuery** - Data querying and extraction
- **Python** (`pandas`, `numpy`, `matplotlib`, `seaborn`) with **Jupyter Notebook:** Data cleaning, analysis, and preliminary visualizations
- **Tableau** - Interactive dashboard design and final visualizations

---

## **Project Files**

| File name                            | Description                                      |
|--------------------------------------|--------------------------------------------------|
| `SaaS_cohort_analysis.ipynb`         | The Jupyter notebook for all analysis steps      |
| `dropoff_by_cohort.csv`              | Cohort-level summary of drop-off data            |
| `README.md`                          | Project documentation                            |

*Note: The full BigQuery export is not included due to large file size.*

---

## **Data Loading, Preparation, and Cleaning**

The dataset was directly loaded into Python from a CSV export of *Google Analytics (BigQuery)*.

Columns imported from BigQuery:
- `fullVisitorId` (renamed as `user_id`)
- `visitId` (renamed as `session_id`)
- `visitStartTime` (renamed as `session_start_time`)
- `hitNumber`(renamed as `hit_number`)
- `hit_type`
- `page_path`
- `revenue`

Initial cleaning steps included:
- Converting `session_start_time` from Unix timestamps to readable dates (datetime format)
- Converting all `hit_type` entries from all caps to lowercase (for readability)

Summary:
- **Rows:** 4,153,675
- **Columns:** 7
- **Unique Users:** 843,049
- **Unique Sessions:** 886,303
- **Missing Values:**
  - `revenue`: 4,141,602 🆗
- **Duplicate Rows:** 0 ✅
- **Date Range:** Aug 1, 2016 - Aug 2, 2017

The `revenue` column will **not** be needed in this particular analysis, so these missing values are irrelevant and can be disregarded.

---

## **Step 1: Purchase Rate by Path**

**Goal:**
Measure how purchase likelihood varies across different navigation paths within the site.

**Key Logic:**
- Found the number of unique navigation paths and identified the most common ones... there were 247,879 unique paths in total
- Calculated how often each path occurs (i.e., frequency)
- Flagged whether a purchase occurred or didn't occur
- Calculated the purchase rate for every unique path (and ultimately filtered for only paths with at least 1 purchase)

**Results:**
Most navigation paths do **not** result in a purchase. It can be inferred that the large majority of paths taken are for general exploration and nothing more; therefore, paths with 0 purchases were filtered out here.

---

## **Step 2: Purchase Rate by Path Length**

**Goal:**
Measure how the length of a user's journey (number of pages viewed during a session) relates to the likelihood of making a purchase.

**Key Logic:**
- Calculated path length (total pages viewed) for every user session
- Counted the number of sessions for every path length
- Created buckets for the path lengths (1, 2, 3, 4-5, 6-10, 11-20, 21-50, 51+ pages viewed)
- Calculated the purchase rate for each bucket

**Results:**
Sessions with **longer journeys** (more pages viewed) are far more likely to result in a purchase than shorter sessions.

---

## **Step 3: Drop-Off Percentage at Each Page Viewed**

**Goal:**
Find out how many user sessions end after each page is viewed during their journey.

**Key Logic:**
- Counted the number of sessions that reached each page view (i.e., page 1, page 2, etc.)
- For each page viewed, calculated how many sessions made it *at least* that far
- Computed the percentage drop-off after each number of page views (e.g., How many sessions, as a percentage, ended after page 7?)

**Results:**
There is an incredibly sharp drop-off at the very beginning. In fact, **over 50%** of user sessions **end after just one page view**. The number of sessions remaining drops significantly with each additional page viewed. While **extensive browsing sessions** are rare, they are much more likely to **result in a purchase**.

---

## **Additional Columns Import from BigQuery**

To improve this analysis (for more significant insights), **four additional columns** were extracted from the BigQuery dataset. The plan was to segment all users into 3-5 **cohorts** that could subsequently be used for comparisons with regard to **how users accessed the site**.

Columns imported from BigQuery:
- `source`: The traffic source responsible for the session (e.g., a google search, dealspotr, facebook, a referral link, etc.)
- `medium`: The marketing channel or medium associated with the session (e.g., "none", organic, referral, affiliate, etc.)
- `pagePath` (renamed to `page_path`): The specific URL or path of the web page being viewed within a session
- `pagePathLevel1` (renamed to `page_path_entry`): The first web page (a.k.a. entry page) a user visited when beginning a session

These columns were merged with the session-level data already present in the Jupyter notebook from the previous analysis steps. Further analysis would determine which column would be used to create cohorts. In other words, which of these columns would dig up the most interesting insights?

---

## **Step 4: Purchase Rate by Source, Medium, Source + Medium, and Entry Page**

**Goal:**
Experiment with the added columns to determine which has the most useful (and interesting) purchase rate trend.

**Key Logic:**
- Calculated the purchase rate for each traffic source
- Calculated the purchase rate for each traffic medium
- Calculated the purchase rate for each combination of traffic source and traffic medium (e.g., dealspotr.com - referral, yahoo - organic, etc.)
- Calculated the purchase rate for each entry page
  - Removed checkout-related pages so more unpredictable results could be highlighted

**Results:**
Purchase rate by traffic medium provided the most useful output. There were only 7 mediums compared to many dozens of types from the other three categories. Plus, the distribution of purchase rates across these 7 mediums had an organized variance. Therefore, it would be best to create 3-5 cohorts from these 7 mediums.

---

## **Step 5: Assign Cohorts by Traffic Medium**

**Goal:**
Categorize every session into a **medium cohort** based on which traffic medium was responsible for how the user accessed the site.

**Key Logic:**
- Defined and ran a conditional function that assigns 7 mediums to 5 cohorts:
  - If `medium` is '(none)'... then the cohort is 'Direct'
  - If `medium` is 'cpm' or 'cpc'... then the cohort is 'Paid'
  - If `medium` is 'organic'... then the cohort is 'Organic'
  - If `medium` is 'referral'... then the cohort is 'Referral'
  - For anything else... then the cohort is 'Other'
- Calculated purchase rate for each of these 5 cohorts

**Results:**
The global average purchase rate is 1.33%. The purchase rates for 'Direct' and 'Paid' are above average, while the rates for 'Organic', 'Referral', and 'Other' are below average.

With all users now properly segmented, more meaningful comparisons can be made.

---

## **Step 6: Drop-Off Percentage at Each Page Viewed**

This is essentially the same as Step 3. The only difference is that a new `cohort` column has been added to that data table so that it is now possible to visualize the drop-off curve for all cohorts.

---

## **Tableau Dashboard**

The Tableau dashboard provides an interactive overview of user journeys across the website.

- **Cohort selection:** A single-value list filter allows viewers to compare traffic medium cohorts across all KPIs and charts. The top left of the dashboard displays a brief description as well as 2-3 key insights.

> Note: The *Other* cohort has been filtered out here because it represents a small number of sessions that could not be properly categorized due to obscure or unknown traffic sources. Including it would just add superfluous statistical "noise". Only **4 cohorts** (plus the sum of all 4) will be utilized from now on.

- **Key performance indicators (KPIs):**
  - The primary metrics are purchase rate, pages viewed per purchase, and pages viewed per exit (without a purchase). These illustrate the main storyline.
  - The secondary metrics are number of purchases, number of sessions, and number of users. These provide supporting context.

- **Chart 1: Purchase rate by journey length:** This line chart demonstrates the simple, yet significant conclusion that the likelihood of a purchase steadily increases as the number of pages viewed during a session increases.

- **Chart 2: User drop-off:** This curve shows how many user sessions remain active as more pages are viewed. Most sessions end rather quickly, but a small number of users continue browsing for many pages, and these particular users are most likely to make a purchase.

---

## **Final Thoughts**

Regardless of how they arrive, most site traffic exits quickly without ever coming close to making a purchase. However, among users who *do* purchase, nearly all took a long, winding journey through the site before any payment was made.

This suggests that the greatest obstacle in the way of a "successful" session isn't always product types or pricing. It also involves getting users to stick around and keep exploring. Perhaps the browsing experience needs to be more engaging, grounding, or rewarding...

Increasing the user base by attracting new traffic is obviously paramount, but a two-pronged approach that also focuses on encouraging deeper engagement from existing users should be considered. From the user's perspective, what would incentivize a person to view *just one more page*? Only one decision can be made at a time. Not ten. Not five. Just one. Multiply those additional clicks across thousands of users, and even rather modest improvements in engagement can result in a meaningful increase in purchases.

---

## **Tableau Dashboard Link**

[User Journey Cohort Analysis (Web Session Data) (Tableau Public)](https://public.tableau.com/app/profile/dylan.barrett1539/viz/UserJourneyCohortAnalysisWebSessionData/Dashboard)
