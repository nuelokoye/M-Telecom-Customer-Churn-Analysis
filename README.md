# M-Telecom-Customer-Churn-Analysis

Telecom Customer Churn Analysis
This telecommunication company provides phone and internet services in california referred to as M telecom.  

Firstly lets define what Churn is? Churn is the measurement of the percentage of accounts/customers who choose not to renew their subscriptions or stop doing business with a company. 
Hence Churn rate is the rate at which customers stops doing business with a company. 

The telecommunication industry is a highly competitive one and i would be carrying out indept analysis to determine the following;
How many customers joined the company during the last quarter? 
What steps can M Telecom take to reduce churn?
What are the key drivers of customer churn?
Is the company losing high value customers? If so, how can they retain them?
A churn profile showing the typical features of a customer susceptible to churn. 

Strategy
Dataset was downloaded from Maven Analytics and it contained information about customer demographics, subscription plans and account records for the Telecom.

As a data analyst understanding the data received is important, and one of the ways to understand your data is by going through the data dictionary. 
 

The main steps for this project are:
Data Cleaning and Preparation
Exploratory Data Analysis
Data Insights
Customer Retention Strategies
Data Visualisation
 

Data Cleaning and Preparation ; 
The dataset was first loaded into excel and cleaned. Firstly the column names had to be renamed appropriately to sql standard by removing the space between the names and replacing them with “_”.  There were alot of empty fields which I had to replace using the Null Value to preserve the integrity of the data for it not to be truncated while loading into MySql Workbench. 
After loading into MySql Workbench, I checked for duplicates in the unique key(Customer_ID) and found none. 

Exploratory Data Analysis ; 
aHow many customers were in the dataset? 

-- Find total number of customers
SELECT
COUNT(DISTINCT Customer_ID) AS customer_count
FROM default_schema.churn

There are 7043 customers in total


How many customers and revenue were lost to churn ?

-- How much revenue did M telecom lose to churned customers?
SELECT Customer_Status, 
COUNT(Customer_ID) AS customer_count,
ROUND((SUM(Total_Revenue) * 100.0) / SUM(SUM(Total_Revenue)) OVER(), 1) AS Revenue_Percentage 
FROM default_schema.churn
GROUP BY Customer_Status;

As shown, the total number of customers is 7,043. Out of which, 1869 are no longer customers. This indicates a 26.54% churn rate (industry standard is 31%). Retention is at a rate of 73.46%, which is quite laudable. While the churn rate is not skyrocket high, the team can take proactive measures to reduce it as 17.2% of the company revenue is lost to churn. 

-- How much revenue has M Telecom lost?
 SELECT 
  Customer_Status, 
  CEILING(SUM(Total_Revenue)) AS Revenue,     -- ceiling is used to Round up the figures--
  ROUND((SUM(Total_Revenue) * 100.0) / SUM(SUM(Total_Revenue)) OVER(), 1) AS Revenue_Percentage
FROM 
  default_schema.churn
GROUP BY 
  Customer_Status;



As seen the 17.2% loss of revenue equated to $3,684,460(approx $3.7 million) 
Average Tenure in Months and Average Monthly Charges of customers:
Tenure is the number of months a customer has been with the company which is shown to be an average of 33 Months while the Average Monthly Charge is $64. 
 
-- Average Tenure in Months and Average Monthly Charges of customers--
SELECT CEILING(AVG(Tenure_in_Months)) As Average_Tenure_in_Months,  CEILING(AVG(Monthly_Charge)) AS Average_Monthly_Charges
FROM default_schema.churn;


What is the Typical Tenure for churners?

-- Typical tenure for churners
SELECT
    CASE 
        WHEN Tenure_in_Months <= 6 THEN '6 months'
        WHEN Tenure_in_Months <= 12 THEN '1 Year'
        WHEN Tenure_in_Months <= 24 THEN '2 Years'
        ELSE '> 2 Years'
    END AS Tenure,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(),1) AS Churn_Percentage
FROM
default_schema.churn
WHERE
Customer_Status = 'Churned'
GROUP BY
    CASE 
        WHEN Tenure_in_Months <= 6 THEN '6 months'
        WHEN Tenure_in_Months <= 12 THEN '1 Year'
        WHEN Tenure_in_Months <= 24 THEN '2 Years'
        ELSE '> 2 Years'
    END
ORDER BY
Churn_Percentage DESC;

This shows that the Majority of Churners which is 41.9% were only with the company for 6 Months or less which is way below the average Tenure of 33 Months.
 Almost half of the customers who churned had a relatively short tenure with the company, so there are opportunities for M Telecom to improve customer retention among newer customers.

Which cities have the highest churn rate ?

Churn rate measures the percentage of customers who stop using the services of a company over a certain period of time.

-- Which cities have the highest churn rates?
SELECT City,
    COUNT(Customer_ID) AS Churned,
    CEILING(COUNT(CASE WHEN Customer_Status = 'Churned' THEN Customer_ID ELSE NULL END) * 100.0 / COUNT(Customer_ID)) AS Churn_Rate
FROM
    default_schema.churn
GROUP BY
    City
HAVING
    COUNT(Customer_ID)  > 30
AND
    COUNT(CASE WHEN Customer_Status = 'Churned' THEN Customer_ID ELSE NULL END) > 0
ORDER BY
    Churn_Rate DESC 
    LIMIT 4; 

These 4 cities have the highest churn rates with San Diego holding a 65% Churn rate which indicates that over half of the San Diego customers have churned. 

5. What were the reasons customers left(I.e Churn Reason)? 
 -- Why did customers leave?
SELECT 
  Churn_Category,  
  ROUND(SUM(Total_Revenue),0)AS Churned_Rev,
  CEILING((COUNT(Customer_ID) * 100.0) / SUM(COUNT(Customer_ID)) OVER()) AS Churn_Percentage
FROM 
  default_schema.churn
WHERE 
    Customer_Status = 'Churned'
GROUP BY 
  Churn_Category
ORDER BY 
  Churn_Percentage DESC;

The major reason most customers stated as their reason for leaving was Competitor, which indicates that M telecom has lost almost $1.7 Million in revenue to competitors. 
More Specific Reason for Customer Churn?
-- Specific Reason why the customers left?
SELECT Churn_Reason, Churn_Category,  
  ROUND(COUNT(Customer_ID) * 100 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage
FROM 
   default_schema.churn
WHERE 
    Customer_Status = 'Churned'
GROUP BY 
  Churn_Reason,
  Churn_Category
ORDER BY 
  Churn_Percentage DESC
LIMIT 5; 


The More specific reasons for which customers left M telecom  for competitors has to do majorly with the competitors having better devices and the competitors making better offers. 
This presents an opportunity to M Telecom to work on their devices, offers and training of their support staff. 

What Promotional offer were Churners on?
-- What offers did churners have?
SELECT
    Offer,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage
FROM
    default_schema.churn
WHERE
    Customer_Status = 'Churned'
GROUP BY
Offer
ORDER BY 
Churn_Percentage DESC;

More than half(56%)  of all churners were not on any promotional offer before leaving which could mean they were either not aware of the promotional offers or the offers were not juicy or Mouth watering enough to keep the customers with M telecom. 

What Internet Type did Churners have? 

-- What Internet Type did churners have?
SELECT
    Internet_Type,
    COUNT(Customer_ID) AS Churned,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage
FROM
    default_schema.churn
WHERE 
    Customer_Status = 'Churned'
GROUP BY
Internet_Type
ORDER BY 
Churned DESC;

66% of Chuners were on the Fiber Optic Internet type. 

What Internet Type did competitors offer churned customers? 
-- What Internet Type did 'Competitor' churners have?
SELECT
    Internet_Type,
    Churn_Category,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage
FROM
    default_schema.churn
WHERE 
    Customer_Status = 'Churned'
    AND Churn_Category = 'Competitor'
GROUP BY
Internet_Type,
Churn_Category
ORDER BY Churn_Percentage DESC;

70% of Churned customers ended up using Fiber Optic with the competitors which indicates that the Fiber Optic Service and quality in M Telecom needs to be reviewed and improved. 

Did Churners have premium tech support? 
-- Did churners have premium tech support?
SELECT 
    Premium_Tech_Support,
    COUNT(Customer_ID) AS Churned,
    ROUND(COUNT(Customer_ID) *100.0 / SUM(COUNT(Customer_ID)) OVER(),1) AS Churn_Percentage
FROM
default_schema.churn
WHERE 
    Customer_Status = 'Churned'
GROUP BY Premium_Tech_Support
ORDER BY Churned DESC;
 
More than 77% of churned customers were not able to access proper technical support with reduced wait time. It is possible that with access to this premium technical support, customers could have improved their after-sales experience and reduced churn.

What Contract were churners on?

-- What contract were churners on?
SELECT 
    Contract,
    COUNT(Customer_ID) AS Churned,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage
FROM 
    default_schema.churn
WHERE
    Customer_Status = 'Churned'
GROUP BY
    Contract
ORDER BY 
    Churned DESC;

Almost all churned customers (89%) were on the month-to-month contract. Customers on a month-to-month contract are more likely to churn, as they have greater flexibility to cancel or switch providers without incurring any penalty.

Which Age bracket did most churners belong to?

-- HOW old were churners?
SELECT  
    CASE
        WHEN Age <= 30 THEN '19 - 30 yrs'
        WHEN Age <= 40 THEN '31 - 40 yrs'
        WHEN Age <= 50 THEN '41 - 50 yrs'
        WHEN Age <= 60 THEN '51 - 60 yrs'
        ELSE  '> 60 yrs'
    END AS Age,
    ROUND(COUNT(Customer_ID) * 100 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage
FROM 
    default_schema.churn
WHERE
    Customer_Status = 'Churned'
GROUP BY
    CASE
        WHEN Age <= 30 THEN '19 - 30 yrs'
        WHEN Age <= 40 THEN '31 - 40 yrs'
        WHEN Age <= 50 THEN '41 - 50 yrs'
        WHEN Age <= 60 THEN '51 - 60 yrs'
        ELSE  '> 60 yrs'
    END
ORDER BY
Churn_Percentage DESC;

32.4% of churners were aged 60 and above. 

Other Characteristics of Churners 
Female Customers who churned were slightly higher(50.2%) than male customers.


 b. Customers who were without dependents were more likely to churn(94.3 %) than customers with dependents. 


c. Customers who are unmarried are more likely to churn (64.2%) as opposed Customers who are married.


d. There was a higher percentage of churn attributed with customers who used phone service than those without phone service. 






e. The Churn attributed to customers with internet service was higher than those without internet service


f. 66% of customers who had no referrals churned. 


14. Key Churn Indicators: 
         The key churn indicators are therefore:
Contract: 89% of churned customers were on the month-to-month contract
Premium Tech Support: 77% of churners did not have premium tech support
Internet Type: 66% of churners used Fiber Optic internet
Offer: 56% of churners did not have any promotional offers, while 23% had Offer E.


15. What amount of High Value customers are susceptible to churn?

-- Are high value customers at risk?

SELECT 
    CASE 
        WHEN (num_conditions >= 3) THEN 'High Risk'
        WHEN num_conditions = 2 THEN 'Medium Risk'
        ELSE 'Low Risk'
    END AS risk_level,
    COUNT(Customer_ID) AS num_customers,
    ROUND(COUNT(Customer_ID) *100.0 / SUM(COUNT(Customer_ID)) OVER(),1) AS cust_percentage
FROM 
    (
    SELECT 
        Customer_ID,
        SUM(CASE WHEN Offer = 'Offer E' OR Offer = 'None' THEN 1 ELSE 0 END)+
        SUM(CASE WHEN Contract = 'Month-to-Month' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Premium_Tech_Support = 'No' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Internet_Type = 'Fiber Optic' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Number_of_Referrals >= 1 THEN 1 ELSE 0 END )AS num_conditions
    FROM 
        default_schema.churn
    WHERE 
        Monthly_Charge > 64 
        AND Customer_Status = 'Stayed'
        AND Number_of_Referrals >= 1
        AND Tenure_in_Months > 6
    GROUP BY 
        Customer_ID
    HAVING 
        SUM(CASE WHEN Offer = 'Offer E' OR Offer = 'None' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Contract = 'Month-to-Month' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Premium_Tech_Support = 'No' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Internet_Type = 'Fiber Optic' THEN 1 ELSE 0 END)+
        SUM(CASE WHEN Number_of_Referrals >= 1 THEN 1 ELSE 0 END)>= 1
    ) AS subquery
GROUP BY 
    CASE 
        WHEN (num_conditions >= 3) THEN 'High Risk'
        WHEN num_conditions = 2 THEN 'Medium Risk'
        ELSE 'Low Risk'
    END; 




I defined high value customers based on the factors below and subsequently grouped them into 3 risk levels (Low, Medium and High):

Tenure: This is a measure of loyalty, so I only considered customers that have been with the company for more than 6 months.

Monthly Charge: Customer’s whose total monthly charge is above the average of $64. 

Referrals: customers who have referred at least one customer to the business.
High-value customers with 3–4 churn indicators are High Risk, while Medium Risk customers have 2 and Low Risk customers have only 1. For instance, a high-value customer at high risk of churning may use fiber optic, have a month-to-month contract, and no promotional offers or premium tech support.

It was observed that out of 1,384 high value customers who stayed, and based on the key churn indicators/drivers, about 60% are at high risk of churning. 

16. Insights
Maven has 1869 churned customers and 20% of them are high-value customers.
42% of churned customers only stayed for 6 months or less.
The top 3 reasons for churn are competitors made better offers, competitors had better devices and attitude of support staff.
Maven lost ~$1.7 million to competitors, making it the most expensive type of churn
The key indicators of churn are Month-to-Month contract , No Premium Tech Support, Fiber Optic internet, No promotional offer and Offer E.
70% of customers who churned to competitors used Fiber Optic
High value customers are churning at a rate of 23%
Based on the key churn indicators, out of 1250 high-value customers remaining, 77% are at high risk of churning.

17. Customer retention strategy 
Loyalty Programs: Since the top reason for churn is ‘competitors making better offers’ , and more than half of churned customers did not have any promotional offers, M Telecom could implement different loyalty programs to retain their customers. For instance, they could reward customers on long-term contracts with discounted rates, free upgrades, or additional features.
Improve Customer Support: Invest in training and development of support staff to ensure they provide excellent customer service. This could include regular coaching and feedback sessions, as well as incentives for staff who receive positive customer feedback.
Make better devices: Evaluate the features, performance and pricing of your devices to ensure they are in line with market standards and demand.
Premium Tech Support: Since customers who did not have access to premium tech support were more likely to churn, M Telecom should consider offering this service to all customers.
Improve Fiber Optic Service: Invest in improving the Fiber Optic service like faster speeds, more stable connections, and better customer support for Fiber Optic customers especially in San Diego and other cities with very high churn rate. 
Engage High-Value Customers: Prioritize engaging these customers to prevent them from leaving. Provide personalized offers, send targeted communications, and provide premium tech support to ensure these customers remain satisfied with their service.
After-Sales Service: Schedule regular check-ins with customers to ensure they are still satisfied with their service. These check-ins could be in the form of surveys, phone calls, or email communications.
18. Final Dashboard( Tableau and Figma):



