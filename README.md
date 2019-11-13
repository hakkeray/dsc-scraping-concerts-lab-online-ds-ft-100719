
# Scraping Concerts - Lab

## Introduction

Now that you've seen how to scrape a simple website, it's time to again practice those skills on a full-fledged site!
In this lab, you'll practice your scraping skills on a music website: https://www.residentadvisor.net.
## Objectives

You will be able to:
* Create a full scraping pipeline that involves traversing over many pages of a website, dealing with errors and storing data

## View the Website

For this lab, you'll be scraping the https://www.residentadvisor.net website. Start by navigating to the events page [here](https://www.residentadvisor.net/events) in your browser.

<img src="images/ra.png">


```python
# Load the https://www.residentadvisor.net/events page in your browser.
```

## Open the Inspect Element Feature

Next, open the inspect element feature from your web browser in order to preview the underlying HTML associated with the page.


```python
# Open the inspect element feature in your browser
```

## Write a Function to Scrape all of the Events on the Given Page Events Page

The function should return a Pandas DataFrame with columns for the Event_Name, Venue, Event_Date and Number_of_Attendees.


```python
import re
import numpy as np
import pandas as pd
import requests
from bs4 import BeautifulSoup
import time
```


```python
r = requests.get("https://www.residentadvisor.net/events/us/losangeles")
soup = BeautifulSoup(r.content, 'html.parser')
events = soup.find('div', id="event-listing")
```


```python
listings = events.findAll('li')
print(len(listings), listings[0])
```

    72 <li><p class="eventDate date"><a href="/events.aspx?ai=23&amp;v=day&amp;mn=11&amp;yr=2019&amp;dy=13"><span>Wed, 13 Nov 2019 /</span></a></p></li>



```python
rows = []
for listing in listings:
    # set current date if it's a date
    date = listing.find('p', class_="eventDate date")
    event = listing.find('h1', class_="event-title")
    if event:
        details = event.text.split(' at ')
        event_name = details[0].strip()
        venue = details[1].strip()
        try:
            n_attendees = int(re.match("(\d*)", listing.find('p', class_="attending").text)[0])
        except:
            n_attendees = np.nan
        rows.append([event_name, venue, cur_date, n_attendees])
    elif date:
        cur_date = date.text
    else:
        continue
df = pd.DataFrame(rows)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>EDIT Room _ Cocodisco, Inbal Lankry and Very S...</td>
      <td>Bar Franca</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>27.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Clinic with Ramon Tapia (Drumcode)</td>
      <td>The Sayers Club</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>7.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Theta Grooves...A Deep/Dub/Minimal Techno Even...</td>
      <td>Gravlax</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Conflict of Interest with Shel—b (Rhythm Rapport)</td>
      <td>Bar Henry</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Made to Move with Kosmik + Stacy Christine</td>
      <td>General Lee's Cocktail House</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>15.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
def scrape_events(events_page_url):
    r = requests.get(events_page_url)
    soup = BeautifulSoup(r.content, 'html.parser')
    listings = events.findAll('li')
    rows = []
    for listing in listings:
        #Is it a date? If so, set current date.
        date = listing.find('p', class_="eventDate date")
        event = listing.find('h1', class_="event-title")
        if event:
            details = event.text.split(' at ')
            event_name = details[0].strip()
            venue = details[1].strip()
            try:
                n_attendees = int(re.match("(\d*)", listing.find('p', class_="attending").text)[0])
            except:
                n_attendees = np.nan
            rows.append([event_name, venue, cur_date, n_attendees])
        elif date:
            cur_date = date.text
        else:
            continue
    df = pd.DataFrame(rows)
    df.head()
    df.columns = ["Event_Name", "Venue", "Event_Date", "Number_of_Attendees"]
    return df
```


```python
scrape_events('https://www.residentadvisor.net/events/us/losangeles')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Event_Name</th>
      <th>Venue</th>
      <th>Event_Date</th>
      <th>Number_of_Attendees</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>EDIT Room _ Cocodisco, Inbal Lankry and Very S...</td>
      <td>Bar Franca</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>27.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Clinic with Ramon Tapia (Drumcode)</td>
      <td>The Sayers Club</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>7.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Theta Grooves...A Deep/Dub/Minimal Techno Even...</td>
      <td>Gravlax</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Conflict of Interest with Shel—b (Rhythm Rapport)</td>
      <td>Bar Henry</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Made to Move with Kosmik + Stacy Christine</td>
      <td>General Lee's Cocktail House</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>15.0</td>
    </tr>
    <tr>
      <td>5</td>
      <td>L.A. 909: Chrome Mamí &amp; Urenium</td>
      <td>Paper Tiger Bar</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>6.0</td>
    </tr>
    <tr>
      <td>6</td>
      <td>Sylvan Esso presents With</td>
      <td>Walt Disney Concert Hall</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td>7</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>8</td>
      <td>Jupiter by Stealth Nights</td>
      <td>TBA - Los Angeles</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>9</td>
      <td>You Know What! 1 Year Anniversary Feat. Floog ...</td>
      <td>TBA - Downtown LA</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>70.0</td>
    </tr>
    <tr>
      <td>10</td>
      <td>Secret Project presents Soul Clap</td>
      <td>Madame Siam</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>24.0</td>
    </tr>
    <tr>
      <td>11</td>
      <td>Technometrik After Hours</td>
      <td>TBA - Downtown LA</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>20.0</td>
    </tr>
    <tr>
      <td>12</td>
      <td>Le Club: Patrice Scott, Aaron Paar</td>
      <td>The Standard, Downtown LA</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>15.0</td>
    </tr>
    <tr>
      <td>13</td>
      <td>Jet Lag 005: J Cush</td>
      <td>TBA - Downtown LA</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>10.0</td>
    </tr>
    <tr>
      <td>14</td>
      <td>Blue Hawaii Live in Los Angeles</td>
      <td>El Cid</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>7.0</td>
    </tr>
    <tr>
      <td>15</td>
      <td>SBCLTR's Intimate Sessions</td>
      <td>Dtla</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>3.0</td>
    </tr>
    <tr>
      <td>16</td>
      <td>Far Away: Buy Music Club with Avalon Emerson &amp;...</td>
      <td>TBA - Los Angeles</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>56.0</td>
    </tr>
    <tr>
      <td>17</td>
      <td>Underline: Miyagi with Contessa and PABLoKEY</td>
      <td>TBA - Los Angeles</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>18</td>
      <td>Sound presents Eli &amp; Fur with Anton Tumas and ...</td>
      <td>Sound</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>19</td>
      <td>Sylvan Esso presents With</td>
      <td>Walt Disney Concert Hall</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>20</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>21</td>
      <td>Breach Christian Martin</td>
      <td>The Basement, Pomona</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>22</td>
      <td>Streo Showcase Feat. Motoe Haus</td>
      <td>Bardot Terrace - Avalon Hollywood</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>23</td>
      <td>Red Bull presents: Tapped In</td>
      <td>The Mayan</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>24</td>
      <td>Floorplay Friday feat. King Felix / Liquorbox</td>
      <td>Station1640</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>25</td>
      <td>Frequency presents Jerm Jerm with Mirage</td>
      <td>Brussels</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>26</td>
      <td>Disclosurefest™</td>
      <td>3 Black</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>27</td>
      <td>Into The Woods with 박혜진 Park Hye Jin</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>117.0</td>
    </tr>
    <tr>
      <td>28</td>
      <td>Factory 93 presents Drumcode Los Angeles</td>
      <td>Exchange LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>64.0</td>
    </tr>
    <tr>
      <td>29</td>
      <td>The Xpander Project: Launch Party</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>29.0</td>
    </tr>
    <tr>
      <td>30</td>
      <td>Cyclone: Afriqua (All Night Long)</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>19.0</td>
    </tr>
    <tr>
      <td>31</td>
      <td>Capsulem: Intimate Night with Nico Stojan (Ouï...</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>16.0</td>
    </tr>
    <tr>
      <td>32</td>
      <td>Technometrik After Hours</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>16.0</td>
    </tr>
    <tr>
      <td>33</td>
      <td>Soft Leather Record Label Launch</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>15.0</td>
    </tr>
    <tr>
      <td>34</td>
      <td>Disconic with Dave Aju, Dead-Tones, AIRS, Conn...</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>12.0</td>
    </tr>
    <tr>
      <td>35</td>
      <td>In Rotation with Momentum</td>
      <td>XL</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>36</td>
      <td>Le Club: Will Saul, Rocky Hill Corps</td>
      <td>The Standard, Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>37</td>
      <td>Defected USA - Los Angeles - Riva Starr, Ferre...</td>
      <td>Sound</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>38</td>
      <td>Moony Habits x Third Place: Bufiman &amp; Francis ...</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>39</td>
      <td>Amazon Uprising, a Party with a Purpose</td>
      <td>Outdoor Stage</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>40</td>
      <td>The Warriors: The Truce, Again</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>41</td>
      <td>Hector Couto &amp; Anthony Attalla</td>
      <td>Station1640</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>42</td>
      <td>Send»return with Jason Douglas, Gold Code, &amp; R...</td>
      <td>The Good Bar &amp; Eatery</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>43</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>44</td>
      <td>Funkbox LA! Tony Touch &amp; Mr. V.</td>
      <td>6555</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>45</td>
      <td>Paradise Boom feat. Dj Lag, Mina, Late Night L...</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>46</td>
      <td>Techlepatic Sessions 040: Anakim &amp; Rivka</td>
      <td>Pattern Bar</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>47</td>
      <td>THE SPACE IN BETWEEN - Facundo Mohrr</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>48</td>
      <td>Frequency presents Dagz Bros with Tyler.Ryyan</td>
      <td>Brussels</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>49</td>
      <td>Disclosurefest™</td>
      <td>3 Black</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>50</td>
      <td>Sunday Sanctuary presents: Jordan Strong, MLE</td>
      <td>One666</td>
      <td>Sun, 17 Nov 2019 /</td>
      <td>3.0</td>
    </tr>
    <tr>
      <td>51</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Sun, 17 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>52</td>
      <td>Rynx</td>
      <td>Fonda Theatre</td>
      <td>Sun, 17 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>53</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Mon, 18 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>54</td>
      <td>Alessandro Cortini</td>
      <td>Lodge Room</td>
      <td>Mon, 18 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>55</td>
      <td>Alessandro Cortini</td>
      <td>Lodge Room</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>56</td>
      <td>Space Yacht 11/19 [Closing Party]</td>
      <td>Sound</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>57</td>
      <td>Temple Tuesdays presents: MD, Bisharat &amp; Jorda...</td>
      <td>Pattern Bar</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>58</td>
      <td>Fade L.A.: Sucia Bonita Latin Bass Night</td>
      <td>La Cita</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>59</td>
      <td>Space Taco Scotty Boy</td>
      <td>The Basement, Pomona</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



## Write a Function to Retrieve the URL for the Next Page


```python
soup.find('a', attrs={'ga-event-action':"Next "}).attrs['href']
```




    '/events/us/losangeles/week/2019-11-20'




```python
def next_page(url):
    r = requests.get(url)
    soup = BeautifulSoup(r.content, 'html.parser')
    link = soup.find('a', attrs={'ga-event-action':"Next "}).attrs['href']
    next_page = "https://www.residentadvisor.net" + link
    return next_page
```

## Scrape the Next 1000 Events for Your Area

Display the data sorted by the number of attendees. If there is a tie for the number attending, sort by event date.


```python
next_page('https://www.residentadvisor.net/events/us/losangeles')
```




    'https://www.residentadvisor.net/events/us/losangeles/week/2019-11-20'




```python
df = scrape_events('https://www.residentadvisor.net/events/us/losangeles/week/2019-11-20')
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Event_Name</th>
      <th>Venue</th>
      <th>Event_Date</th>
      <th>Number_of_Attendees</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>EDIT Room _ Cocodisco, Inbal Lankry and Very S...</td>
      <td>Bar Franca</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>27.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Clinic with Ramon Tapia (Drumcode)</td>
      <td>The Sayers Club</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>7.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Theta Grooves...A Deep/Dub/Minimal Techno Even...</td>
      <td>Gravlax</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Conflict of Interest with Shel—b (Rhythm Rapport)</td>
      <td>Bar Henry</td>
      <td>Wed, 13 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Made to Move with Kosmik + Stacy Christine</td>
      <td>General Lee's Cocktail House</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>15.0</td>
    </tr>
    <tr>
      <td>5</td>
      <td>L.A. 909: Chrome Mamí &amp; Urenium</td>
      <td>Paper Tiger Bar</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>6.0</td>
    </tr>
    <tr>
      <td>6</td>
      <td>Sylvan Esso presents With</td>
      <td>Walt Disney Concert Hall</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td>7</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>8</td>
      <td>Jupiter by Stealth Nights</td>
      <td>TBA - Los Angeles</td>
      <td>Thu, 14 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>9</td>
      <td>You Know What! 1 Year Anniversary Feat. Floog ...</td>
      <td>TBA - Downtown LA</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>70.0</td>
    </tr>
    <tr>
      <td>10</td>
      <td>Secret Project presents Soul Clap</td>
      <td>Madame Siam</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>24.0</td>
    </tr>
    <tr>
      <td>11</td>
      <td>Technometrik After Hours</td>
      <td>TBA - Downtown LA</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>20.0</td>
    </tr>
    <tr>
      <td>12</td>
      <td>Le Club: Patrice Scott, Aaron Paar</td>
      <td>The Standard, Downtown LA</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>15.0</td>
    </tr>
    <tr>
      <td>13</td>
      <td>Jet Lag 005: J Cush</td>
      <td>TBA - Downtown LA</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>10.0</td>
    </tr>
    <tr>
      <td>14</td>
      <td>Blue Hawaii Live in Los Angeles</td>
      <td>El Cid</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>7.0</td>
    </tr>
    <tr>
      <td>15</td>
      <td>SBCLTR's Intimate Sessions</td>
      <td>Dtla</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>3.0</td>
    </tr>
    <tr>
      <td>16</td>
      <td>Far Away: Buy Music Club with Avalon Emerson &amp;...</td>
      <td>TBA - Los Angeles</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>56.0</td>
    </tr>
    <tr>
      <td>17</td>
      <td>Underline: Miyagi with Contessa and PABLoKEY</td>
      <td>TBA - Los Angeles</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>18</td>
      <td>Sound presents Eli &amp; Fur with Anton Tumas and ...</td>
      <td>Sound</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>19</td>
      <td>Sylvan Esso presents With</td>
      <td>Walt Disney Concert Hall</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>20</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>21</td>
      <td>Breach Christian Martin</td>
      <td>The Basement, Pomona</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>22</td>
      <td>Streo Showcase Feat. Motoe Haus</td>
      <td>Bardot Terrace - Avalon Hollywood</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>23</td>
      <td>Red Bull presents: Tapped In</td>
      <td>The Mayan</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>24</td>
      <td>Floorplay Friday feat. King Felix / Liquorbox</td>
      <td>Station1640</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>25</td>
      <td>Frequency presents Jerm Jerm with Mirage</td>
      <td>Brussels</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>26</td>
      <td>Disclosurefest™</td>
      <td>3 Black</td>
      <td>Fri, 15 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>27</td>
      <td>Into The Woods with 박혜진 Park Hye Jin</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>117.0</td>
    </tr>
    <tr>
      <td>28</td>
      <td>Factory 93 presents Drumcode Los Angeles</td>
      <td>Exchange LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>64.0</td>
    </tr>
    <tr>
      <td>29</td>
      <td>The Xpander Project: Launch Party</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>29.0</td>
    </tr>
    <tr>
      <td>30</td>
      <td>Cyclone: Afriqua (All Night Long)</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>19.0</td>
    </tr>
    <tr>
      <td>31</td>
      <td>Capsulem: Intimate Night with Nico Stojan (Ouï...</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>16.0</td>
    </tr>
    <tr>
      <td>32</td>
      <td>Technometrik After Hours</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>16.0</td>
    </tr>
    <tr>
      <td>33</td>
      <td>Soft Leather Record Label Launch</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>15.0</td>
    </tr>
    <tr>
      <td>34</td>
      <td>Disconic with Dave Aju, Dead-Tones, AIRS, Conn...</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>12.0</td>
    </tr>
    <tr>
      <td>35</td>
      <td>In Rotation with Momentum</td>
      <td>XL</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>36</td>
      <td>Le Club: Will Saul, Rocky Hill Corps</td>
      <td>The Standard, Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>37</td>
      <td>Defected USA - Los Angeles - Riva Starr, Ferre...</td>
      <td>Sound</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>38</td>
      <td>Moony Habits x Third Place: Bufiman &amp; Francis ...</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>39</td>
      <td>Amazon Uprising, a Party with a Purpose</td>
      <td>Outdoor Stage</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>40</td>
      <td>The Warriors: The Truce, Again</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>41</td>
      <td>Hector Couto &amp; Anthony Attalla</td>
      <td>Station1640</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>42</td>
      <td>Send»return with Jason Douglas, Gold Code, &amp; R...</td>
      <td>The Good Bar &amp; Eatery</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>43</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>44</td>
      <td>Funkbox LA! Tony Touch &amp; Mr. V.</td>
      <td>6555</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>45</td>
      <td>Paradise Boom feat. Dj Lag, Mina, Late Night L...</td>
      <td>TBA - Downtown LA</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>46</td>
      <td>Techlepatic Sessions 040: Anakim &amp; Rivka</td>
      <td>Pattern Bar</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>47</td>
      <td>THE SPACE IN BETWEEN - Facundo Mohrr</td>
      <td>TBA - Los Angeles</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>48</td>
      <td>Frequency presents Dagz Bros with Tyler.Ryyan</td>
      <td>Brussels</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>49</td>
      <td>Disclosurefest™</td>
      <td>3 Black</td>
      <td>Sat, 16 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>50</td>
      <td>Sunday Sanctuary presents: Jordan Strong, MLE</td>
      <td>One666</td>
      <td>Sun, 17 Nov 2019 /</td>
      <td>3.0</td>
    </tr>
    <tr>
      <td>51</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Sun, 17 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>52</td>
      <td>Rynx</td>
      <td>Fonda Theatre</td>
      <td>Sun, 17 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>53</td>
      <td>AFI Fest 2019 presented by Audi</td>
      <td>TCL Chinese Theatres</td>
      <td>Mon, 18 Nov 2019 /</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>54</td>
      <td>Alessandro Cortini</td>
      <td>Lodge Room</td>
      <td>Mon, 18 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>55</td>
      <td>Alessandro Cortini</td>
      <td>Lodge Room</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>56</td>
      <td>Space Yacht 11/19 [Closing Party]</td>
      <td>Sound</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>57</td>
      <td>Temple Tuesdays presents: MD, Bisharat &amp; Jorda...</td>
      <td>Pattern Bar</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>58</td>
      <td>Fade L.A.: Sucia Bonita Latin Bass Night</td>
      <td>La Cita</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>59</td>
      <td>Space Taco Scotty Boy</td>
      <td>The Basement, Pomona</td>
      <td>Tue, 19 Nov 2019 /</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



## Summary 

Congratulations! In this lab, you successfully developed a pipeline to scrape a website for concert event information!
