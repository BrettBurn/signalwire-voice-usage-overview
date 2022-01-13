# signalwire-voice-usage-overview
Script to generate a summary of a given SignalWire Project's voice usage. Utilizes SignalWIre's List Calls API. 

# Overview 

In this snippet, we will show you how to use SignalWire's List Calls API to gain valuable insight into your usage of voice on SignalWire's platform. After running this script, you gain both itemized and bird's eye views on all of the details of your voice usage, including:

* The "To" and "From" number.
* The time the call started.
* The call SID.
* The price of the call.
* The call direction.
* Whether the call was answered by a human or an answering machine. 
* The call status.

While this snippet will provide you with a lot of information, it can easily be manipulated to discover more! Take a look at our [API Documentation] to find out more. 

# What do I need to run this code? 

As we will be using Python for this script, you will need to install the [SignalWire Python SDK](https://developer.signalwire.com/compatibility-api/reference/client-libraries-and-sdks#ruby). Once installed, you will also need your SignalWire API credentials, including an **API Token, SignalWire Space URL, and a Project ID**. All of this information can be found under the **API** tab of your SignalWire Space, however, you can find more information on where to look [here](https://developer.signalwire.com/apis/docs/navigating-your-space).

You will need to have a few different Python Packages installed as well to achieve this. 
* [DateTime](https://docs.python.org/3/library/datetime.html) for creating an ability to filter by a specific time frame. 
* [Pandas](https://pandas.pydata.org/) for creating data frames and making it easier to manage large quantities of data.


# Code Breakdown

The first thing we'll do is import all of the resources we spoke about earlier and instantiate the client. From there, we call the List Calls API as you can see below. Notice that `datetime` is called and uses the format `datetime(YYYY, MM, DD)`. This tells the script over what time period you wish to examine and gain insight from this project. 

```python
from datetime import datetime
from signalwire.rest import Client as signalwire_client
import pandas as pd

client = signalwire_client("ProjectID", "AuthToken", signalwire_space_url='example.signalwire.com')

calls = client.calls.list(start_time_after=datetime(2021, 10, 1), start_time_before=datetime(2021, 10, 31))
```

Once we complete that, next we're going to create an array and loop through all of the calls based on the desired time frame we set earlier and then append any additional data that we wish to use as it's looping. We then categorize the status of the calls in order to quantify the calls in various statuses. 

```python
d = []
status = []

for record in calls:
    d.append((record.from_, record.to, record.start_time, record.sid, record.price, record.direction, record.answered_by, record.status))
    status.append(record.status)

total_queued=int(status.count("queued"))
total_ringing=int(status.count("ringing"))
total_in_progress=int(status.count("in progress"))
total_canceled=int(status.count("canceled"))
total_completed=int(status.count("completed"))
total_busy=int(status.count("busy"))
total_failed=int(status.count("failed"))

num_outbound_calls = int(calls.count("outbound"))
num_inbound_calls = int(calls.count("inbound"))
```

Next we'll use the toolset given to us by installing Pandas to create a data frame that will organize this information into a much more manageable and readable format. Notice the line `df.to_csv('Calls.csv', index=False, encoding='utf-8')`. This line exists to export the itemized data frame that this snippet provides you with to a CSV file. After, we total up the individual prices of each call in order to provide a real-time total price that resulted from your Voice API usage. 

```python
df = pd.DataFrame(d, columns=('From', 'To', 'Start Time', 'Call SID',  'Price', 'Call Direction', 'Answered By', 'Call Status'))
print(df)
df.to_csv('Calls.csv', index=False, encoding='utf-8')
print()

totalCalls = len(df)
totalCost = df['Price'].sum()
formattedCost = "${:,.2f}".format(totalCost)
```

The final portion of this snippet prints a more holistic summary of what your Voice API usage has been based off the dates you've provided to the List Calls API, giving you insight of the total price of the calls, the total number of calls inbound and outbound, and how those calls performed. 

```python
print("You sent " + str(totalCalls) + " total calls during your selected date range.")
print("The total cost of calls in your selected date range is approximately " + formattedCost + " USD.")
print("There are currently " + str(total_queued) + " calls in the queue, with " + str(total_ringing) + " currently ringing and " + str(total_in_progress) + " currently in progress.")
print("There have been " + str(total_canceled) + " calls that have been canceled.")
print("There have been " + str(total_completed) + " completed calls, " + str(total_busy) + " calls marked as busy, and " + str(total_failed) + " failed calls.")
print("There were " + str(num_inbound_calls) + " inbound calls and " + str(num_outbound_calls) + " outbound calls.")
```

If you wish to see the itemized table that gives you full detail into every call, simply search for the CSV file on your computer that exported as a result of that script. 

