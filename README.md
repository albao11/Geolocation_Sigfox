
Contributors: Alba Ordoñez, Karine Petrus, Ioan Catana, Stéphane Mulard

# Geolocation of messages sent by connected devices

## Context 
The purpose of this work consisted of finding the exact position of a set of messages sent by connected devices that could be or not in motion. This information had to be retrieved from data received by Sigfox stations which captured the messages. 

Two datasets were provided:

- A training set consisting of features associated to the messages and the true location of each message.
- A test set on which we applied the prediction model, but for which we did not have the real positions. It was on this set that we were evaluated.

## Summary of the implemented ideas

We splitted the provided training set into two parts:

- A training dataset
- A validation dataset that allowed us to check the performance of our predictions on messages for which we knew the exact positions.
To unsure that our final predictions were good and unbiased, the characteristics of the validation dataset had to reflect as closely as possible the characteristics of the final test set on which we were evaluated. 
Consequently, we made sure to create a validation set with no devices present in the training set (this was one important feature of the final test set).

For the feature matrix used for predictions, we created a table so that each message was associated with the list of stations that received it together with new features described below:

- d_keep: limit of distance to keep, expressed according to the rssi (measuring the received signal strength)
- newrssi: penalty introduced at rssi level
- bs_lng_centroid: centroid of longitudes of stations having seen the same message
- bs_lat_centroid: centroid of latitudes of stations having seen the same message
- bs_count_mess: giving more weight to the most powerful stations
- distance: distance between the message and the station that received it (calculated using the Haversine formula).

The last feature (distance) proved to be very useful to reduce errors made in geolocation. 
However, calculating this distance required an estimate of the latitudes and longitudes of the messages. 
Note that the latter are in fine the quantities we seek to calculate. 
In the case of our validation dataset, we already have the ground truth for latitudes and longitudes, 
but this is not the case for the final test set on which we were evaluated. 
Therefore, we chose to proceed with the calculation of the message positions as if we did not know
the ground truth. More specifically, we used a 2 iterations procedure:

- During the first iteration: a rough estimate of the latitudes and longitudes of the messages was obtained. 
This estimate allowed us to calculate the distance feature. Then, we could also correct the position of the abnormally located stations.
- During the second iteration: first the positions of the new corrected stations were used to calculate the 
features bs_lng_centroid and bs_lat_centroid. Then, we prepared our feature matrix 
(including d_keep, newrrsi, bs_count_mess, etc.) to predict more accurately the message positions.

The prediction of the final locations was performed using the Extra Trees Regressor of scikit-learn. On our validation dataset, we obtained an error cumulative probability at 80 % of 2800 meters.

The code and a detailed analysis of the data and our methodology to compute the predictions is given in the jupyter notebook: **TP_geoloc.ipynb**.

An HTML export of the notebook can be found in **TP_geoloc_VF.zip**. This zip file contains an HTLM visualization of the notebook together with image files.
