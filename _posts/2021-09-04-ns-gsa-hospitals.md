---
classes: wide
title: "Exploring the Accessibility of COVID-19 Referral Hospitals in Metro Manila"
date: 2022-02-28
last_modified_at: 2024-08-18
permalink: /projects/covid19-hospitals
tags: [COVID-19, geospatial analysis, network science]
excerpt: "Exploring COVID-19 Referral Hospitals in Metro Manila using Network Science and Geospatial Analysis"
category: "projects"
mathjax: "true"
show_date: "true"
---
*This is a combination of my final project for Network Science with Sasha Garcia and Introduction to Geospatial Analysis with Vee Rivera.*

## Executive Summary
The COVID-19 pandemic emphasized the Philippines' vulnerable healthcare system - especially for the COVID-19 referral hospitals. With the government’s commitment to democratize healthcare in the country, they need to either increase the capacity or build more healthcare facilities. In a 2015 [article](https://businessmirror.com.ph/2015/
12/01/big-investors-see-hospitals-as-good-business/), the expense of putting up a hospital is estimated to cost around $9M. Taking into account inflation, the cost of building one today will cost even more. Hence, building more healthcare facilities without analyzing the current service area of existing facilities will be costly.

As such, we aim to assess the accessibility of the COVID-19 referral hospitals. We approach this problem by performing a service area analysis of these facilities in Metro Manila, building a bipartite network, performing simulation to identify the capacity of each, and solving a facility location problem to propose optimal location for new facilities. 

As a result, we were able to identify unserved barangays which are not within a certain walking distance from a referral hospital. We were also able to calculate for node redundancy using the bipartite network and able to identify less redundant hospitals that may need additional support. Additionaly, our simulation provided metrics that would assist in assessing the needs of these facilities. Lastly, we are able to determine the optimal sites to establish new healthcare facilities to increase the coverage in Metro Manila.

<iframe src="\assets\images\project\ns-gsa-hospitals\isochrone_map_5.html" width="100%" height="400px" style="border:none;"></iframe><p style="font-size:12px;font-style:default;"><b>Figure 1. Coverage Map of Hospitals (COVID-19 referral hospitals) in Metro Manila.</b><br> Isochrone maps are generated per facility per trip time (5-minute, 15-minute, 30-minute, 45-minute, 60-minute)</p>

## Data 
__1. Global Administrative Area Database (GADM)__
The Global Administrative Area Database (GADM) contains spatial data for countries in varying levels of detail. For this study, we focused on the barangay level, the lowest level of administrative division in the Philippines.

__2. Philippine Census__
The Philippine Census contains data of the population on a barangay level. The most recent available data (2015 Census Data) was used in this study.

__3. DOH COVID-19 Tracker__
The DOH COVID-19 Tracker is an open data initiative by the Department of Health as part of their efforts to fight against the pandemic as well as to promote public health. It contains COVID-19 related data such as number of new cases, recoveries, and deaths. The granularity of data is on a barangay level. In addition to this, it includes data on the COVID-19 referral hospitals which was used for this project.

## Methodology
In this section, we will be discussing the methodology pipeline implemented in this project. A high-level overview is outlined as follows:

<img src="\assets\images\project\ns-gsa-hospitals\ns-methodology.png" height="100" width="85%">

<p style="font-size:12px;font-style:default;"><b>Figure 2. Methodology Pipeline.</b><br>Five main steps were done in order to address the problem statement and obtain the results.</p>

### Data Processing
Using the data discussed in the Data Section, the data was processed and prepared for analysis. In this subsection, we will be discussing how the extracted data were processed before performing the service area analysis, network analysis, simulation, and solving facility location problem.

#### Barangays in Metro Manila
With the data obtained from the Philippine Census, we have identified $1,691$ barangays in Metro Manila with a total population of $~13$ million.

<img src="\assets\images\project\ns-gsa-hospitals\polygons.png" width="40%">
<p style="font-size:12px;font-style:default;"><b>Figure 3. Barangays in Metro Manila.</b><br> The barangays in Metro Manila were plotted using extracted polygons from the GADM dataset.</p>

Additionally, since we want to simulate a demand point, we assumed that most of the population of a barangay is found in its center.
<img src="\assets\images\project\ns-gsa-hospitals\centroid.png" width="40%">
<p style="font-size:12px;font-style:default;"><b>Figure 4. Barangays Centroids in Metro Manila.</b><br> The barangays centroids in Metro Manila were plotted using extracted polygons from the GADM dataset.</p>

#### Road Network of Metro Manila
Performing the service area analysis would require the road network of Metro Manila. To extract this, we used the Python package `osmnx` which allows users to extract geospatial data from OpenStreetMap.

<img src="\assets\images\project\ns-gsa-hospitals\road-network.png"  width="40%">
<p style="font-size:12px;font-style:default;"><b>Figure 5. Road Network of Metro Manila.</b><br> Approximately $117,000$ intersections were used as nodes and $319,000$ streets were used as edges to define the network.</p>

#### Building Footprints of Makati City
Aside from the barangay centers as a demand point, we also want to have a more granular level of detail. Hence, we assumed that a building can also be a demand point. As such, we extracted building footprints. Since extracting building footprints for the entire Metro Manila will require a huge computing power, we limited our search to the city of Makati as a proof-of-concept. In extracting the building footprints in Makati City, we also used the Python package `osmnx`.

<img src="\assets\images\project\ns-gsa-hospitals\mktbf.png" width="30%">
<p style="font-size:12px;font-style:default;"><b>Figure 6. Extracted polygon of building footprints in Makati City.</b><br> Approximately $55,295$ building footprints were used to simulate a demand pont.</p>

To simply our analysis, we extracted each building’s centroid.

<img src="\assets\images\project\ns-gsa-hospitals\mktbfpoint.png" width="30%">
<p style="font-size:12px;font-style:default;"><b>Figure 7. Building footprints in Makati City with a precision = $3$.</b> The precision of decimal degrees allow us to unambiguously recognize neighborhood and streets, retaining important information for the analysis.</p>

#### Referral Hospitals in Metro Manila
Using the data from the DOH COVID-19 tracker, we were able to scrape the locations of the referral hospitals using [Google Places](https://developers.google.com/maps/documentation/places/web-service/overview) API. There are a total of 159 COVID-19 referral hospitals in Metro Manila with 874 ICU beds, 4,795 isolation beds, and 3,772 ward beds.

<img src="\assets\images\project\ns-gsa-hospitals\referral-hospitals.png" width="40%">
<p style="font-size:12px;font-style:default;"><b>Figure 8. COVID-19 referral hospitals in Metro Manila.</b><br>The location of the facilities in Metro Manila were identified and plotted on top of the the extracted polygons from the GADM dataset.</p>

## Service Area Analysis
Using the road network, we are able to visualize how accessible the referral hospitals are. Here, we used the walk network type as this is the most accessible to the general population with the standard speed of walking set at $4$km/hr at different walking distances. This enables us to see the differences in accessibility given increasing walking duration. Assumptions that were used in order to perform the service area analysis are summarized in Table 1.

<p style="font-size:12px;font-style:default;"><b> Table 1. Service Area Analysis Assumptions </b></p>
<table>
  <tr>
    <th style="font-size:12px">Parameter</th>
    <th style="font-size:12px">Assumption</th>
  </tr>
  <tr>
    <td style="font-size:12px">Network Type</td>
    <td style="font-size:12px">Walk</td>
  </tr>
  <tr>
    <td style="font-size:12px">Walking Speed</td>
    <td style="font-size:12px">4 km/hr</td>
  </tr>
  <tr>
    <td style="font-size:12px">Trip Time</td>
    <td style="font-size:12px">5-, 15-, 30-, 45-, 60-min</td>
  </tr>
</table>

Once the service areas are generated, we can identify barangays that are not accessible per service area. To perform this, isochrone maps are generated to show areas accessible from a point within the trip times identified in Table 1. To generate isochrone maps, a combination of Network Science and Geospatial Analysis techniques were implemented. Using the extracted road network of Metro Manila, we implemented the algorithm below, which is based on a publicly available example of the usage of [OSMNx](https://github.com/gboeing/osmnx-examples/blob/v0.13.0/notebooks/13-isolines-isochrones.ipynb).

<div style="display: flex; justify-content: center;">
    <img src="\assets\images\project\ns-gsa-hospitals\algorithm.jpg" width="60%">
</div>

A total of $795$ isochrone maps were generated per facility per threshold. Figure 9 shows the results for each trip time. Intuitively, a longer trip time will translate to a larger service area.
<img src="\assets\images\project\ns-gsa-hospitals\isochrones.png" width="100%" height="250px">
<p style="font-size:12px;font-style:default;"><b>Figure 9. Coverage Map of COVID-19 Referral Hospitals in Metro Manila.</b><br> Isochrone maps are generated per hospital per trip time (From L to R: $5$-minute, $15$-minute, $30$-minute, $45$-minute, $60$-minute).</p>

## Hospital-Barangay Network
To create a hospital-barangay bipartite network, we assign the hospitals as the first set of nodes and the barangays as the second set of nodes. Nodes of the hospitals connect directly to the nodes of the barangays and there are no direct hospital-to-hospital or barangay-to-barangay connection. Figure 10 illustrates this network.

<img src="\assets\images\project\ns-gsa-hospitals\bipartite-network.png" width="30%">
<p style="font-size:12px;font-style:default;"><b>Figure 10. Hospital-Barangay Bipartite Network.</b><br> A bipartite network was used to represent the relationship between hospitals and barangays.</p>

Here, a connection between a hopsital and barangay is established if the barangay is within $n$-minutes away from the hospital, with $n$ representing the different trip times. By using the results from the Service Area Analysis, we also generated $5$ bipartite networks based on varying trip times.
<img src="\assets\images\project\ns-gsa-hospitals\bipartite-triptime.png" height="250px" width="100%">
<p style="font-size:12px;font-style:default;"><b>Figure 11. Hospital-Barangay Bipartite Network.</b><br>Connections between the hospitals and barangays gradually become more evident as trip times increase.</p>

### Capacity Simulation
Once the bipartite network has been established, we are able to use the network to simulate a COVID-19 scenario wherein patients from a barangay will seek treatment from hospitals within its network as defined by the bipartite network. As such, we can follow the progress of the capacity of the hospitals and the number of unserved cases through time. In the simulation, we use the $30$-minute bipartite network established in the Hospital-Barangay Network subsection and assume a random assignment of hospitals for each patient of a barangay within its hospital-barangay bipartite network. The assumptions are summarized in Table 2:

<p style="font-size:12px;font-style:default;"><b> Table 2. Service Area Analysis Assumptions </b></p>
<table>
  <tr>
    <th style="font-size:12px">Parameter</th>
    <th style="font-size:12px">Assumption</th>
  </tr>
  <tr>
    <td style="font-size:12px">% of Susceptible Individuals in a Barangay</td>
    <td style="font-size:12px">$10%$</td>
  </tr>
  <tr>
    <td style="font-size:12px">Probability that an Individual will Contract COVID-19</td>
    <td style="font-size:12px">$1%$</td>
  </tr>
  <tr>
    <td style="font-size:12px">Probability that an Infected Person will need to be Hospitalized</td>
    <td style="font-size:12px">$1.5$</td>
  </tr>
  <tr>
    <td style="font-size:12px">Hospital Capacity</td>
    <td style="font-size:12px"># of ICU beds + isolation beds + ward beds</td>
  </tr>
  <tr>
    <td style="font-size:12px">No. of Days in Hospital</td>
    <td style="font-size:12px">$14$ days</td>
  </tr>
  <tr>
    <td style="font-size:12px">Days of Simulation</td>
    <td style="font-size:12px">$30$ days</td>
  </tr>
</table>

In the simulation, we assign the size of the node to represent the hospital's current capacity. Throughout the simulation, the nodes will increase its size to indicate that there have been an increasing number of patients coming into that hospital while the opposite will mean a decrease in patients in the hospital. Aside from this, a
green node means that the hospital is still within capacity, while a red node means that the hospital is already at full capacity.

### Facility Location Problem
The Facility Location Problem is concerned about the specific question: "How do we place our hospitals, clinics and other healthcare facilities such that the maximum number of people are within its coverage?" Such a problem fall under the Maximum Coverage Location Problem (MCLP). Which, under the assumption that the hospital sites are
coordinated ahead of time, solves the optimal sites to place healthcare facilities to cover the greatest number of unserved population.

While there are many ways to solve an MCLP, including complex methods such as kernel density estimation (KDE) or
least significant difference (LSD), the simplest way is to solve it randomly. This is precisely the method used by Can Yang in their MCLP module `mclp.py` found in their [Github repository](https://github.com/cyang-kth/maximum-coverage-location).

The algorithm is indeed simple, and loosely operates as follows:
Given parameters:
1. $M$ - Number of candidate sites
2. $R$ - Radius of coverage of sites
3. $K$ - Number of sites

*Loosely* execute as follows:
1. Generate $M$ number of candidate sites in random locations in the vicinity of the concerned population
2. Examine all permutations of $K$ number of sites and check for their total coverage with radius $R$
3. Select the permutation with the greatest coverage of the concerned population

We selected $M = 1000$ candidate sites, which is more than enough for the concerned population. With
this, the optimal sites do not make any significant changes, and consistently cover the same population between repeat runs.

Additionally, since we are essentially solving a resource allocation problem, we are also somewhat bound as to the
number of sites, $K$, that we use. Ideally, we want to find the maximum coverage with minimum number of sites used.

## Discussion of Results
In this section, we present the results of the service area analysis, hospital-barangay bipartite network, and the network simulation.

### Service Area Analysis
As discussed in the Road Network of Metro Manila subsection, the isochrone maps show the areas serviced by the referral hospitals. Through this, we have identified the barangays which are not within a service area of the hospitals.
<iframe src="\assets\images\project\ns-gsa-hospitals\unserved_barangays.html" width="100%" height="300px" style="border:none;"></iframe>
<p style="font-size:12px;font-style:default;"><b>Figure 12. Unserved Barangays.</b><br>Each trip time shows unservered barangays in Metro Manila.</p>

The results of the service area analysis are quantified in Table 3. Intuitively, the areas covered by referral hospitals increase as the trip time increase, it follows that the number of unserved barangays gradually decrease. Results also show that the unserved population decrease as the trip time increase.
<p style="font-size:12px;font-style:default;"><b> Table 3. Service Area Analysis Results </b></p>
<table cellpadding="5" cellspacing="0">
  <tr>
    <th style="font-size:12px">Walking Duration</th>
    <th style="font-size:12px">$5$ mins</th>
    <th style="font-size:12px">$15$ mins</th>
    <th style="font-size:12px">$30$ mins</th>
    <th style="font-size:12px">$45$ mins</th>
    <th style="font-size:12px">$60$ mins</th>
  </tr>
  <tr>
    <td style="font-size:12px">Unserved Barangays</td>
    <td style="font-size:12px">$1,228$</td>
    <td style="font-size:12px">$453$</td>
    <td style="font-size:12px">$37$</td>
    <td style="font-size:12px">$6$</td>
    <td style="font-size:12px">$3$</td>
  </tr>
  <tr>
    <td style="font-size:12px">Unserved Population</td>
    <td style="font-size:12px">$6.7$M</td>
    <td style="font-size:12px">$2.6$M</td>
    <td style="font-size:12px">$238$K</td>
    <td style="font-size:12px">$25$K</td>
    <td style="font-size:12px">$3$K</td>
  </tr>
</table>

### Hospital-Barangay-Network
Intuitively, as the trip time increases, the number of connected hospitals and barangays increase as well. In addition to this, an important aspect of the hospital-barangay network is that we are able to calculate, on the average, the number of hospitals accessible per barangay and the number of barangays in the service area of a hospital. The results for all trip times are quantified in Table 4.
<p style="font-size:12px;font-style:default;"><b> Table 4. Service Area Analysis Results </b></p>
<table cellpadding="5" cellspacing="0">
  <tr>
    <th style="font-size:12px">Walking Duration</th>
    <th style="font-size:12px">$5$ mins</th>
    <th style="font-size:12px">$15$ mins</th>
    <th style="font-size:12px">$30$ mins</th>
    <th style="font-size:12px">$45$ mins</th>
    <th style="font-size:12px">$60$ mins</th>
  </tr>
  <tr>
    <td style="font-size:12px">Average # of Hospitals Accessible per Barangay</td>
    <td style="font-size:12px">$1$</td>
    <td style="font-size:12px">$2$</td>
    <td style="font-size:12px">$5$</td>
    <td style="font-size:12px">$10$</td>
    <td style="font-size:12px">$16$</td>
  </tr>
  <tr>
    <td style="font-size:12px">Average # of Baranggays in the Service Area of a Hospital</td>
    <td style="font-size:12px">$4$</td>
    <td style="font-size:12px">$17$</td>
    <td style="font-size:12px">$55$</td>
    <td style="font-size:12px">$108$</td>
    <td style="font-size:12px">$171$</td>
  </tr>
</table>

Another important aspect of the hospital-barangay bipartite network is the ability to calculate for node redundancies. In essence, nodes with high redundancy have less impact to the overall network in the event that it is removed from the network. On the other hand, nodes that have a low redundancy bring more impact to the network if it is removed. Focusing on the $30$-minute bipartite network, we see that there are multiple nodes that have a low node redundancy. It is important to identify these as a starting point for resource allocation.
<iframe src="\assets\images\project\ns-gsa-hospitals\hospital_network_redundancy.html" width="100%" height="430px" style="border:none;"></iframe><p style="font-size:12px;font-style:default;"><b>Figure 13. Node Redundancy.</b><br>The identification of less redundant nodes help highlight which nodes have high impact to the network when removed.</p>

### Capacity Simulation
Simulating the capacity of the referral hospitals given the assumptions stated in Table 4 has allowed us to capture metrics such as number of unserved cases per day and the number of hospitals at full capacity. Aside from this, we can also identify which hospitals have the most number of days at full capacity. We believe that
these metrics will help asses the need to provide extra healthcare facilities in order to avoid overloading the hospitals, thus being able to accommodate all COVID-19 patients at any given time.
<iframe src="\assets\images\project\ns-gsa-hospitals\simulation_results.html" width="100%" height="430px" style="border:none;"></iframe><p style="font-size:12px;font-style:default;"><b>Figure 14. 30-Day Capacity Simulation of the Hospitals.</b><br>The simulation shows which hospitals are within capacity (green nodes) and which reached full capacity (red nodes).</p>

Based on the simulation, the number of unserved cases have an increasing trend since Day 1. However, Figure 15 shows that there are also evident peaks that reflect surges in COVID-19 cases. Through this simulation, hospitals will be able to anticipate peaks or surge of cases so that they can better prepare and manage resources.

<img src="\assets\images\project\ns-gsa-hospitals\unserved-cases.png" width="50%">
<p style="font-size:12px;font-style:default;"><b>Figure 15. Number of Unserved Cases per Day.</b><br>Using the number of unserved cases per day as a metric will help anticipate peaks or surges to better manage resources.</p>

Determining the number of hospitals at full capacity per day is a great indicator of how much healthcare facilities are lacking in NCR. Through this metric, we will be able to assess the need to provide healthcare facilities in order to ensure that hospitals will not reach their full capacity and close themselves off to potential patients. Results of this metric are illustrated in Figure 16.

<img src="\assets\images\project\ns-gsa-hospitals\unserved-cases.png" width="50%">
<p style="font-size:12px;font-style:default;"><b>Figure 16. Number of Hospitals at Full Capacity.</b><br> Using the number of hospitals at full capacity as a metric will help assess the need to provide extra healthcare facilities.</p>

Lastly, it is also important that we are able to identify which hospitals are at full capacity and the corresponding days they remained at that status. Figure 17 shows the top $20$ hospitals that have the most number of days at full capacity. This can help in assessing the need to increase the hospitals capacity or to add other
healthcare facilities within the area of these hospitals.
<img src="\assets\images\project\ns-gsa-hospitals\days-full.png" width="55%">
<p style="font-size:12px;font-style:default;"><b>Figure 17. Number of Days at Full Capacity per Hospital.</b><br>Some hospitals are seen unable to recover given a certain number of days of being at full capacity.</p>

### Facility Location Problem
For the Facility Location Problem, we focused on the results of the $30$-minute threshold. Figure 18 shows the unserved location of barangays and building footprints. These are the demand points identified wherein we will try to recommend optimal location for new healthcare facilities which will be discussed in the succeeding section.

<img src="\assets\images\project\ns-gsa-hospitals\service-areas.png" width="50%">
<p style="font-size:12px;font-style:default;"><b>Figure 18. Unservered Areas Used for Facility Location Problem.</b><br><b>L:</b>Unservered barangays in Metro Manila and <b>R:</b> Unserved building footrpints in Makati City.</p>

#### Demand Point: Barangay
Shown in Figure 19 are the covered barangays vs hospital sites generated by the MCLP algorithm. The graph shows
diminishing returns. By the 2<sup>nd</sup> hospital placed, the number of covered barangays is already ~$2/3$ of the total number of unserved barangays. Any hospitals placed beyond the second show significantly reduced coverage, with $8$ additional hospitals completely covering all the unserved barangays.
<img src="\assets\images\project\ns-gsa-hospitals\ncrgraph.png" width="50%">
<p style="font-size:12px;font-style:default;"><b>Figure 19. Covered Barangays vs Hospital sites in the MCLP.</b><br>The coverage starts to diminish at $2$ sites.</p>

The result of the MCLP algorithm for the first $4$ optimal sites is shown in Figure 20.
<img src="\assets\images\project\ns-gsa-hospitals\mclp_brgy.png" width="100%" height="300px">
<p style="font-size:12px;font-style:default;"><b>Figure 20. Optimal Sites from MCLP using Barangay as Demand.</b><br>Optimal sites by solving MCLP in <span style="color:green;">GREEN</span>, unserved barangays are in <span style="color:red;">RED</span>, and existing referral hospitals in <span style="color:purple;">PURPLE</span>.</p>

#### Demand Point: Building Footprints
A similar result is shown in the scale of Makati. Shown in Figure 21 is the coverage for $1$ to $4$ sites. This time, the population is much denser, so there is some inevitable overlap between sites. Notice how the initially placed sites shift when another is placed to avoid overlap. This is an inherent result of the algorithm as any overlap takes away maximization of the covered area since redundancy is not considered in the algorithm.
<img src="\assets\images\project\ns-gsa-hospitals\mclp_bldg.png" width="100%" height="280px">
<p style="font-size:12px;font-style:default;"><b>Figure 21. Optimal Sites from MCLP using Building Footprints as Demand.</b><br>Optimal sites by solving MCLP in <span style="color:green;">GREEN</span>, unserved barangays are in <span style="color:red;">RED</span>, and existing referral hospitals in <span style="color:purple;">PURPLE</span>.</p>

Now, shown in Figure 22 are the covered buildings vs hospital sites generated by the MCLP algorithm. Similar to the NCR scale, the graph shows diminishing returns, albeit at a lesser rate. By the 4<sup>th</sup> hospital, the coverage is already $ ~ 4/5 $ of the total number of buildings. The lesser rate of diminishing returns is a result of the more densely packed population, as well as there being a greater number of population or buildings to consider. Even still, by the 9<sup>th</sup> hospital/clinic placed, all the buildings are essentially served save for a few outliers.
<img src="\assets\images\project\ns-gsa-hospitals\mktgraph.png" width="50%">
<p style="font-size:12px;font-style:default;"><b>Figure 22. Covered Buildings vs Hospital sites in the MCLP.</b><br>The coverage starts to diminish at $4$ sites.</p>

## Conclusion
With the continuous increase in COVID-19 cases in the Philippines, especially in the National Capital Region, it is important to bring focus to the facilities that help treat those COVID-19 infected individuals. By identifying referral hospital and their service areas, we were able to identify barangays that are not within n-minutes from
COVID-19 referral hospitals. This allows us to further highlight the needs of the communities who are extremely challenged in our current situation.

Aside from this, using a hospital-barangay bipartite network, we were able to calculate node redundancies and highlight less redundant hospitals that may need additional support, be it additional resources or additional healthcare facilities in their area. By identifying these hospitals, we may be able to create short-term
and long-term solutions that will further aid in addressing not only the current pandemic, but also future similar situations.

Using our simulation, we were able to capture metrics that can help our government make policies in terms of allocating resources. As we highlight hospitals that are mostly at full capacity, we believe that these can serve as triggers to assess the capability of hospitals to accommodate the communities around them and open up the opportunity to bring even more resources to communities and barangays that need them.

Lastly, we explored the uncovered areas and determined the optimal sites for new facilities. This was done through a Maximum Coverage Location Problem (MCLP) on two scales: the NCR-scale with barangay centroids as the demand points and Makati-scale with building centroids as the demand points. On both scales, the coverage vs number of sites showed diminishing returns, with the NCR scale thresholding by the 2<sup>nd</sup> site, and Makati scale thresholding by the 4<sup>th</sup> site.