# Covid-Predictor-Germany
Using LSTM to predict mean number of COVID cases in the next week in a district, based on the data from previous week. The data is used from fusionbase.io

## Title

This is an attempt to predict the number of COVID cases in an area in the near future. These numbers are important to take precautionary safety measures and allocate necessary resources in order to tackle an adverse situation. For example, the area that's predicted to have an increased number of cases can be pre-supplied with proper medical equipment.

## Intuition

I have used LSTM based model for making the predictions. LSTM works great with sequential data. The intuition was pretty basic. Say, in an area X, the number of COVID cases have been recorded as 5, 10, 12, 18, ...100 and so on. We want to find (predict) the next number in the sequence. As the disease is contagious, the sequence takes almost a monotonically increasing form. However, we cannot have an exact mathematical function to predict the number. Thus deep learning comes to play, which by its inherent black-box nature, tries to estimate what the next number could be. 

## Data

 I have used the COVID data for [Germany](https://public.fusionbase.io/explore/covid19-germany/data). This data informs us about the district wise number of COVID cases in Germany. The data is in a timeseries format (ideal for LSTM), and multiple updates are made to the database each day.

## The Process

### Obtaining the raw data

There are multiple ways to get the data. You can download the data as a whole csv file, or you can make API calls. In order to save coding and testing times I opted for the former. After obtaining the entire dataset that spanned over 1-1.5 months, I had to trim the dataset, because it had multiple entries for same days, and I needed only the latest data for every day. This was done by sorting the data by timestamp of entry, and grouping by date(day) and obtaining the latest data for each day.

### Data pre-processing

####  Label Encoding

I had decided to use **area_names** and **bundesland_names** as features, because by intuition it is evident that certain places are better at dealing with the pandemic than the others. However, to feed this categorical data to the model I had to perform Label Encoding. I kept the Encoders for future use.

#### Scaling

**population, number of cases, cases per 100k** were also important features in the dataset, but they needed scaling before we could feed them to the model. I used MinMaxScaler() for the same, to bring down the values between 0 and 1. I also kept the Scalers for future use.

####  Obtaining sequential data (for training and testing)

Now we need sample points for the model. Let's think of each point as (X,y) where X is the input and y is the output. Each input point is a sequence of COVID data in a particular area for 7 days, and each output is the mean of COVID case numbers in that area for the next 7 days. Thus, we are asking the model a question -> "Hey, if I tell you the sequence of how was the situation last week, can you tell me the mean (average) number of cases next week?". 

### The LSTM model

I used 4 LSTM layers (CuDNNLSTM since they exploit the NVIDIA GPU and training is faster) with 128 units each, interspaced with 20% percent Dropout layers to prevent overfitting. I used the 'Adam' optimizer and 'Mean square logarithmic loss' for the model compilation and trained the model for 100 epochs. These numbers are obviously Hyper-parameters, and if I had more time I would have done a Grid Search to obtain the best set of parameters.

### Results

I obtained an R2 score (R2 score is preferred for regression problems) of more than 90 percent. I then used the trained model for predicting the mean number of cases in the future in an area by feeding it the latest data available for the area. **The model is making reasonable predictions for areas that are moderately affected by COVID in this particular dataset (example Darmstadt, Stuttgart etc.). Its predictions suffer when dealing with the areas with extreme case numbers like Berlin or Munich.** This can be further worked upon. 

## Future plans

I would also like to merge the data with weather data to see what effects weather changes have on our predictions, and identify if weather may play a role in covid numbers.

Then I would also like to check with mobility data (movements of cars/taxis/people) to identify how big a role social distancing is actually playing in controlling covid and what drastic steps are actually not needed and what steps can be granted to ease a bit of freedom of movement of the people during this hard time.
