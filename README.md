# Predicting Airbnb Prices in New York

Federico Anzil - February 2020


## Introduction - Business Problem

Airbnb allows hosts to rent their properties to Airbnb users. Host set their own prices per night. Price per night is the most important variable for defining the hosts’ revenue. While **Airbnb hosts** can experiment with different prices to find the ‘best price’ for their properties, new hosts can have a hard time in finding their optimal price, resulting in loses because of days with too high prices and low occupation, or too low prices and a hidden opportunity cost.

The objective of this project is to create a predictive model of Airbnb prices to solve this problem. Also, model outcome will sever as a basis to better understand pricing determinants or be used for finding low price opportunities by **Airbnb guests**.

Using this models, we will be able to predict the price of a property taking into account features like room types, number of bathrooms, number of bedrooms, how many people is allowed, and so on.

Another objective is to find how do near venues influence pricing. For instance, how will be the price influence is the property is near a coffee shop, a museum or a transport hub? By incorporating these features, our model will be useful for predicting how will the price change after the introduction or closure of a nearby venue. 


## The Data

While Airbnb doesn’t release public data, the private webpage named [Inside Airbnb](http://insideairbnb.com/get-the-data.html) releases data obtained from scraping the official Airbnb website. Inside Airbnb webpage contains several datasets for large cities of the world, including New York City. Latest datapoint at the moment of writing is from 04 December, 2019.

For each city, we find the following datasets:

- listings: detailed listing data
- calendar: calendar data
- reviews: review data

For our model, we will be using only the listing data. This data contains 106 columns. Some of the columns I will be focusing are:

- Price
- Latitude
- Longitude
- Accommodates
- Review Scores
- Room Type
- Amenities
- Extra People
- Bathrooms
- Bedrooms


![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1580937571220_data_head.png)


For data related to near venues, I will use the Foursquare Developer API, that allows us to get data like category and rating from near venues, passing a latitude and longitude.

You can see a sample of the data that can be extracted using the Foursquare API in the following image:

![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1580938060257_venues.png)


As we can see, we can get the venue location and category. 


## Methodology

The Airbnb dataset contained 106 columns but for this analysis only the ones that I considered more important were kept. Some variables, are not related with this analysis, like the URL or the id. For other variables, like the listing summary, or the description of the space, could be included using other methods out of the scope of this report. 

After this initial cleaning, the dataset consisted on 22 features, including latitude and longitude. The location information is important, because we will use this information to merge this dataset with data from Foursquare, to take into account the venues near the listing. I expected that the most common kind of venues where a listing is located could play a role in the price. 

**Venues Clustering**

Using the Fourquare API and a , I created 5 clusters to differentiate neighborhoods according to the most common venues. 

For example, in cluster 1 we have relatively more pizza places, banks and pharmacies. Cluster 2 has tons on Italian restaurants, bars, bodegas and cafes. Cluster 3 has more parks and farmer markets; and so on. 

Using the Folium library, I created a map to visualize clusters:

![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1580996066663_nyc_cluster_map.png)


As we can see, there are two big clusters and 3 smaller ones. Further analysis on the topic could focus only on certain areas where Airbnb is more popular (like Manhattan) and take into account venues that could be more important to tourists, like transport hubs, museums and parks. Another approach could take into account venues density or distance to important venues instead. 

After running our model, I appended the cluster label to the generated venues dataset, that contains information of venues with latitude, longitude and cluster. This allowed me to merge this data with the Airbnb dataset, and add cluster data to each Airbnb listing.


![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1580998149578_nyc_cluster_map_2.png)


Most of Airbnb properties are located on clusters 1 and 2:


    cluster_2    36641
    cluster_1     7139
    cluster_4       77
    cluster_0       63
    cluster_3       21

**Data Preparation**

**Logaritmic Transformation**

Looking at the distribution of the price (our target variable) we see that a logarithmic transformation could be better to achieve a normal distribution. 

![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1580998466907_price-dist-1.png)


So, I applied a logarithmic transformation to the price all the non-dummy numerical features. Afterwards, this is the distribution of the price:


![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1580998580883_price-dist-2.png)


**Data Cleaning and Prepossessing**

Some of the steps of the data cleaning involved:

- Removing listing with price = 0
- Replacing outliers with price too low (<$10) or too high(>$1000)
- Filtering amenities: I took into account only a subset of amenities, that include security deposit, if extra people is allowed, cleaning fee, air conditioning, parking on premises, and so on. As we will see below, cleaning fee is an important determinant of the price. 

For prepossessing, clusters and amenities were converted to dummy variables.

You can find all the cleaning and prepossessing steps on the Jupyter Notebook. 

**Modelling** 

I used a Gradient Boosting method from the XGBoost library. This algorithm gained popularity because of in the data science world because of it’s ability to come up with very accurate models on a wide array of business situations. Feel free to find more about XGBoost and how to use it at https://www.datacamp.com/community/tutorials/xgboost-in-python


## Results

The model result was able to achieve an r**2 of 0.70

Here is what I found:

As expected, the most important determinant of the price was the feature ‘room_type_Entire home/apt’ , followed by the neighborhood and the number of bathrooms. 


## 
![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1580999646362_xboost-1.png)


Of course, comparing different properties according if it is the entire home/apt vs shared properties is like comparing oranges and apples, so afterwards I focused only on 

This time, the R**2 lowered to 0.49 but we have more meaningful information from the model.

Taking into account only entire home/apartments, the most important features are number of bathrooms, neighborhood (location), washer machine and cluster(location).

![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1581000242444_features-2a.png)


And here is the decision tree:


![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1581000561461_tree.png)


In the following plot  we can see a visualization of the accuracy of the model. In this plot, logarithmic scales were converted back to the original scale:

![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1581000351562_accuracy.png)

## Discussion

Here we can see the relation between bathrooms and price (only for 1 or 2 bathrooms, and whole apartment or house):


![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1581000988425_bathrooms.png)


Relation between neighborhood and price:

![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1581001126459_neigborhood.png)


And the cluster:

![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1581001365289_cluster.png)


And washer:

![](https://paper-attachments.dropbox.com/s_B9867FD161A8BE904DEDC97DB0387E383785BD478243FD8943B5D2EB7DC11A70_1581001415103_washer.png)


As we can see, the model allows us to find some important features to predict the price, besides if the property is shared or not. 

Interestingly, and unexpectedly for me, reviews scores didn’t play such a big role. Neither did the number of people that can be accommodated in a property. 

Most important features were number of bathrooms, bedrooms and washer amenity. 

Location did play a big role. Prices in Manhattan and in cluster 2 tend to be higher. 


## Conclusions and Recommendations

The XGBoost model was able to explain half of the price variation given the features. Some important features were location related, like neighborhood and other were amenities like washer machine and cleaning fee. 

Some variables not included in this study could add more explanation power to the model. 

Future similar modelling should take seasonality into account. 

Also, as we could see, location and near venues seem to play a role. It would be interesting to include related variables like distance to important features like museums, airports and so on. And also include other location variables like crime rate per neighborhood. 
