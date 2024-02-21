For our dataset, we wanted to see if there was any correlation between a countrys
median age and the age of its leader. We decided that a scatterplot would be the 
best way to show this, since we are looking at two continuous variables. 

Since there are so many points, it would be extremely confusing to label them all 
at once on the graph, so we decided to add interaction and create a mouseover 
tooltip that shows each country. We thought the graph was initially pretty boring 
and lacked color, so we decided to create a color scheme based around country reigons. 
Since we did this, we decided that we could make the visualization more interesting and 
interactive by letting the viewer filter data by region. 

We had some other ideas that we scrapped along the way. The first one was that of 
adding interactive trendlines. However when we took a look at the data, we realized 
that there really wasn't a major trend, median_age and leader_age appeared to have 
little correlation. 

In terms of splitting up the work, David - I created the scatterplot visualization 
in App.svelte and setup the interaction functionality. I'd say I spent around 10 hours 
on this, but the majority of the time was trying to figure out how to use d3 and integrate 
it with the github. After fixing technical errors and looking through d3 and svelte 
documentation and videos, creating the visualization and interaction itself took 
considerably shorter.