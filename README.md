# Hotel Booking Demand Exploratory Analysis
Using Spark to analyze Hotel Booking Demand

## Data Source & Background
The data used in this analysis is from Kaggle, however the raw data comes from this science article Hotel booking demand datasets. The dataset records 119,390 bookings made to 2 hotels in Portugal, one city hotel in Lisbon, and one resort hotel in Algarve, between 1st of July of 2015 and the 31st of August 2017. 

The original data was pulled from the PMS (property management system). Excluding the hotel column, it has 31 attributes. These attributes can be segmented into 4 categories:

* Time related columns: any column that is time-bound, e.g. arrival_date_year, arrival_date_month...
* Guest information related columns: any column that could describe the person making the order, e.g. country, adults, children, babies, is_repeated_guest...
* Order related columns: the columns that are not guest-specific, e.g. market_segment, distribution_channel, reserved_room_type...
* Issue related columns: any column that is related to cancelation, e.g. is_cancelled, previous_cancellation...

## Goal of the Analysis
The purpose of this analysis is to better understand the correlation between the attributes of the order and cancellation. For example, some of the questions that this analysis aims to answer are:
* Does the group size of the guests influence cancelation?
* Are there any specific nationalities of guests more likely to cancel?
* Does the demographic of the guests of these two hotels differ?
* Do the guests who have previously cancelled more likely to cancel again?

### Further topic to explore
Cancelations are undesirable, but No-Shows (customers did not check-in and did not inform the hotel of the reason why) are even more troublesome for hotels, because if with enough time in between arrival & cancelation, hotels are more likely to resell the rooms. 
The focus of this analysis is on cancelation, however a deep dive on what correlates with No-Show could be beneficial & practical to hotel revenue management. 

## Approach 
PySpark and its SQL function and SQL type packages are the tools that’s being used for this analysis. 
### 1. Data Cleaning & Basic Profiling 
When the data was originally read into the Spark environment, it had no missing value. However, after investigation, in the column of agent, company and children there are a few null values which were encoded incorrectly for PySpark, so transformation was done on these columns. Several time related columns are also not encoded in the desired datetime format, they were also transformed.
Then the columns are grouped by their unique values to count their frequencies, so the composition of the data could be better understood. In the process, the distribution of their respective values shows, so it could be used to decide if this column has distinguishing power.  Because if the majority of a column’s values are heavily concentrated on value only, it does not give us additional information.
### 2. Create New Columns & Segments 
Based on the understanding gained from the previous step, a list of new columns is created. For the binned columns, the thresholds are set according to their statistics & frequencies (and some common sense for explainability). 

* total_headcount: adults + children + babies
* guest_size: bin the total_headcount into 4 categories: solo, couple, small_group, big_group
* is_family: whether there are children or babies in the order or not
* is_agent: if this booking comes via a travel agent
* is_company: whether this booking is paid by a company or not
* adr_cat: bin the adr column into 3 categories: below_average, average, above_average
* lead_time_cat: bin the lead_time into 4 categories: same_day, short, medium, long
* total_stays: stays_in_weekend_nights + stays_in_week_nights
* stay_length_cat: bin the total_stays into 5 categories: one, two, three, medium, long
* booking_change_cat: bin booking_changes into 4 categories: none, one, some, many
* season_cat: whether this order’s arrival date is during high season, low season or other

The dataset is also broken down into two to have a more granular understanding of each hotel. For example, The adr_cat column needs to be hotel-specific, because each hotel has a different baseline for price. 

### 3. Comparison Across Dimensions
With the metrics created above, cancelation rate can be compared across  many different dimensions:

* Guests profile regarding family & group size
* Booking through Company & customer type
* Distribution channel, market segment & agents
* Nationality
* Meals
* Lead time & number of changes made the booking
* Lead time & Price
* Lead time & deposit
* Previous cancelation
* Seasonality

## Conclusion
### Insights

* Group & Transient group orders have much lower cancelation rate across both hotels, while contract orders, though small, have a high cancellation rate especially for leisure travellers (not paid by companies) in the city hotel. 
* Non-family couple travellers (cannot assume relationship, just mean it’s a party size of 2) are the most likely to cancel for the city hotel, maybe due to the fact that they are the biggest composition in terms of demographic for this hotel. For the resort hotel, the most likely to cancel are the non-family big groups, though they are little in number of orders. The more significant demographic which is prone to cancel is small families with children.
* Orders that come from travel agents are more likely to cancel for both hotels. The lists of the agents who have contributed to high cancellation rates & sizable orders are provided in the notebook, and the lists for each hotel differs. 
* Offline travel agents have lower cancellation rates for the resort hotel, while not so for the city hotel. The segmentation of Groups is bringing sizable orders for both hotels but also has a higher than average cancelation rate.
* Lists of the nationality of the guest with higher cancellation rates & sizable orders have been listed in the notebook, while again the lists are quite different for each hotel, while both have domestic guests as most likely to cancel, due to domestic travellers being the majority for both hotels. 
* Guests who have booked full-board meal plans are highly prone to cancelation, though not big in size for both hotels. 
* The longer the lead time is, the more likely the order is to be canceled, which is very natural.  However, if the cancelation gap is large enough, there are high chances that the hotel could resell the rooms.
* If the guests made just 1 or a few changes to the booking, it has a much lower cancelation rate.  Whereas if the guests made either no or many change(s), they are more likely to cancel.
* For the resort hotel, the higher the average daily rate is, the more likely the order will be cancelled. However, for the city hotel in the couple segment, it is not the case. The super long lead time with a below average adr combination has a cancellation for 84%, which is much higher than the baseline (41%).
* For the non-refundable orders, curiously for both hotels, have a very high cancellation rate (99% for city hotels and 95% for resort hotels). Further attributes were considered but cannot seem to find a confounding factor. 
* For both hotels, orders arriving in the low season (month of February, November, December and January) have lower cancellation rates, especially more apparent for the resort hotel.
* Those who have canceled previously one time are very likely to cancel again for both hotels. However for the city hotel, those who have canceled 2 or 3 times are actually less likely to cancel compared to a new guest. 

### Recommendation
Based on the insights gained the analysis, two strategies are recommended:
1. Check up with the guest 2 weeks prior to the arrival date
As seen in the booking changes breakdown, guests who have made 1 or few changes are the least likely to cancel. Therefore the hotels could contact the guests beforehand (e.g. 2 weeks before) if there's any changes that shall be made, maybe those who want to cancel would cancel then, and hence cause less no-show and give the hotel more time to resell the room.
2. Overbook when orders have high-cancellation traits 
The nationality, size of the group, meal plans and agents are all indicators of how likely these orders will be cancelled. When the demand is high and the rooms are fully booked, the hotel revenue management team could look into the composition of the orders and decide if they should increase supply. 
