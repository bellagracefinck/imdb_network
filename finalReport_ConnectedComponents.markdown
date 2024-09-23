### Network Analysis Final Project
Kaitlyn Crowley, Bella Grace Finck, and Anna Glass
May 14, 2024

# Introduction

The movie industry is driven by vast and interwoven networks of
collaboration between directors and the crews. This paper presents a
network analysis that explores the patterns and structures underlying
creative collaborations in film, leveraging data from the Internet Movie
Database (IMDb) for a list of 101 directors and their crew members. In
this paper, we outline our process to extract the IMDb data, construct a
network of director and crew nodes, visualize illustrative trends, and
compute statistics for analysis to explore three central research
questions:

1.  How have directors influenced the careers of crew members? What
    directors have had the most influence on crew members?

2.  How do the roles of crew members fluctuate?

3.  How widespread is the phenomenon of directors re-using the same
    crew?Do renowned directors (and women/minority directors) tend to
    work persistently with the same key collaborators compared to lesser
    recognized directors or a random Hollywood director?

This network analysis contributes to a deeper understanding of
collaboration dynamics, career and role mobility, and the impacts of
close collaborations to understand the network underpinning film
production.

# Data Extraction 

The data extraction task of this project involved two primary steps
after we were provided with the code to scrape the IMDb database for the
desired data. The output of this step is a directory containing a folder
labeled with the ID of each director of interest. Within each director's
folder, there is a .json file containing the director's information and
data for each movie they've directed and that movie's associated cast.

``` {#lst:terminalInputs .python language="Python" caption="Running code in terminal example." label="lst:terminalInputs"}
python crowley_imdb_scraper.py --parent-directory '/Users/kaitlyncrowley/Desktop/Spring2024/DATA445/FinalProject/film_directors/' --director-data-path '/Users/kaitlyncrowley/Desktop/Spring2024/DATA445/FinalProject/100_film_directors.csv' --director-id-input nm0336620,nm0000229
```

## Data Normalization

In the data normalization process, we normalized the variants of
\"Cast\" and \"Writing Credits\". Specifically, we renamed \"Cast (in
credits order)\", \"Cast (in credits order) complete, awaiting
verification\", \"Cast (in credits order) verified as complete\", and
\"Cast complete, awaiting verification\" to their root role of \"Cast.
Similarly, we normalized \"Writing Credits (WGA)\", \"Writing Credits
(WGA) (in alphabetical order)\", and \"Writing Credits (in alphabetical
order)\" to their root role of \"Writing Credits\". We completed this
normalization by calling the \"normalize_movie_role\" function within
the \"filter_credit_roles\" function.

In addition to normalizing the major roles, we also normalized the
subroles of various crew members working on the movies. Many of these
roles contained extra information that was unnecessary for our analysis
and added extra noise to the data. To complete this normalization, we
called the \"normalize_movie_subrole\" within the
\"filter_credit_roles\" function. This function returned a \"normalized
credit\" for each crew member who had a subrole, which maintained the
integrity of the original data in case future users of the data or code
wanted to access it.

## Data Filtering

While the process of normalizing the data removed much of the
unnecessary granularity in the data, we also completed a filtering step.
In this step, we filtered out all crew members except for the ones with
the following roles: \"Casting by\", \"Cinematography by\", \"Costume
Design by\", \"Directed by\", \"Film Editing by\", \"Music by\",
\"Production Design by\", \"Writing Credits\", \"Makeup Department\",
\"Produced by\", \"Sound Department\", and \"Special Effects by\". By
only including these roles, we significantly decreased the number of
nodes in the network, making it easier to manage in Gephi and making it
easier to interpret the patterns in the network.

Within each of these filtered roles, some crew members had specific
titles for their subroles within their department. For example, there
were many different subroles in the sound department, including roles
such as \"foley artist\", \"boom operator\", \"chief sound engineer\",
and \"dialogue editor\". However, for this project, we only wanted to
focus on a few of the key subroles. For the sound department, we
filtered out all crew members except those whose normalized credit
(computed in the normalization process) was \"re-recording mixer\",
\"sound designer\", or \"supervising sound editor\". Some crew in the
sound department held multiple roles after normalization (see Figure
[1](#fig:soundRoles){reference-type="ref" reference="fig:soundRoles"}).
For the \"Produced by\" role, we only included the crew members whose
normalized credit was \"producer\", \"producer produced by\", or
\"producer produced by p.g.a.\". We also filtered out subroles for the
\"Makeup Department\", only including crew members whose normalized
credit was \"hair department head\", \"makeup department head\", or
both. This part of the filtering was tricky because, for some crew
members, their \"normalized credit\" contained two roles. Finally, we
filtered out any crew members whose role was listed as \"Special Effects
by\" except for the crew whose normalized credit was \"special effects
supervisor\" or \"visual effects supervisor\". For the other roles, we
kept all of the associated crew members.

![Example of a crew member with multiple
sub-roles.](img/soundRoles.png){#fig:soundRoles width="0.5\\linewidth"}

It is important to note that this process of normalization and filtering
is not perfect, in large part due to the inconsistencies in IMDb data.
In future research, it would be interesting to investigate how this data
is recorded, who records it, and what guidelines are in place to manage
the quality and accuracy of the data that is available.

## Data Download

Using the CSV file provided that contained the information for the 101
directors of interest, I was able to loop through each IMDb uri and
extract the desired data, filtering and normalizing when necessary. It
is important to note that, if the director had a lot of associated data,
the download would timeout, so I integrated a method of rerunning the
download for those problem IDs. This allowed me to download all 101
directors, only running the code once.

The result was a nested dictionary for each director that contained
general information about the director, including their ID, race,
gender, and whether they were within the top 20 highest-grossing
directors in the dataset. Each director dictionary contained a key
\"complete_credits\". The associated value was another dictionary
containing each of the movies the director directed and a nested
dictionary containing information about the individual crew members who
worked on each movie (see Figure
[\[directorSubset\]](#directorSubset){reference-type="ref"
reference="directorSubset"}). In addition to the crew information, each
movie dictionary contained an \"imdb_details\" key that contained
additional details about the movie, including a description, the content
rating, and a summary of the movie reviews.

![Extract from Steven Spielberg's JSON file. This image captures the
data for one crew member, Tony Kushner, for one of the films Spielberg
directed, \"The Fabelmans\".](directorSubset.png){#fig:directorSubset
width="0.5\\linewidth"}

# Network Generation {#network-generation .unnumbered}

## Network Structure {#network-structure .unnumbered}

Our network model is made up of two distinct node types, **director
nodes (D)** and **crew nodes (C)**, each possessing different
attributes, and two distinct edge types, director-director edges (D-D)
and director-crew edges (D-C). The network itself is an undirected,
weighted graph.

## Nodes {#nodes .unnumbered}

**Director nodes** possess the following attributes (see Figure 1 in the
Appendix for a detailed table of director node attributes):

-   role

-   sex

-   race

-   labels (if applicable)

-   total_films

-   collabs

![An example output showing director node
attributes.](director_node.png){#fig:dir_node width="0.5\\linewidth"}

Whereas **crew member nodes** possess the following attributes (see
Figure 2 in the Appendix for a detailed table of crew node attributes):

-   roles

-   subroles

-   directors

-   years

-   total_films

-   total_directors

-   min_year

-   max_year

-   start_role

-   end_role

-   num_roles

-   num_subroles

-   role

![An example output showing crew node
attributes.](crew_node.png){#fig:crew-node width="0.5\\linewidth"}

## Links {#links .unnumbered}

If a director and crew member worked together on a film, the two nodes
are linked. Similarly, if a director worked under another director in a
non-directing role, the two directors are linked. The only link type
that does not exist in our network is crew-crew links. These could be
discovered, however, were we to create a projection of the network
linking together crew members if they worked with the same director.

**Director-Director links (D-D)**: Links between directors are weighted
by the number of times the directors collaborated on films OTHER THAN
co-directing (Ex: When Director A is a producer on a film Director B is
directing, their edge weight increases by one. The edge is undirected,
so it will also increase by one if Director B is a producer/writer/other
non-director role on a film Director A is directing.)

![An example of a director-director link.](DD_link.png){#fig:dd_link
width="0.25\\linewidth"}

**Director-Crew links (D-C)** possess two edge attributes:
'collaboration_score' and 'weight'.

1.  Collaboration score is the simpler of the two, defined as the
    percentage of the director's career (in \# of films) spent with a
    specific crew member. It is calculated by dividing the total number
    of collaborations between the director and the crew member (# of
    times director's name is mentioned in the crew member node's
    'directors' attribute) by the total number of films directed by the
    director ('total_films" attribute in director node)

    1.  *collaboration_score = (total collaborations **÷** total films)
        \* 100%*

    2.  For a director who has directed 4 films and worked with a
        specific crew member 2 times, the link between the two nodes
        will have a collaboration_score of 50%.

2.  The **weight** of the edge is similar to the collaboration score
    metric, but normalized to account for the significance of specific
    crew member roles. The calculation uses the same numerator as in the
    collaboration_score metric (the total number of collaborations).
    However, rather than dividing by the total number of films directed
    by the director, we divide by the total number of times the director
    worked with someone in the same role as the crew member (including
    the crew member him or herself, however many times they worked with
    the director).

    We use this as the edge weight rather than the collaboration score
    or a raw count of total collaborations because certain crew roles
    are more time-consuming or rare than others. For example, there are
    usually considerably fewer writers on a film than there are members
    of the Sound Department, so the weights between directors and
    writers are generally greater than the weights between directors and
    sound crew.

    1.  *weight = (total collaborations **÷** role frequency) \* 100%*

    2.  If the crew member's primary role is producer and they've worked
        with the director 2 times, the total_collaborations value is 3.
        If the director has directed 3 films with 1 producer, 3
        producers, and 4 producers respectively, the role frequency
        value is equal to 1 + 3 + 4 = 8. Therefore, the weight is equal
        to 2/8 = 1/4.

![An example of a director-crew link.](DC_link.png){#fig:dc_link
width="0.5\\linewidth"}

## Assumptions and Justifications {#assumptions-and-justifications .unnumbered}

1.  The crew member's primary role is defined as the role they possessed
    most frequently over the course of their career. This is justified
    by the fact that the vast majority (97.4%) of crew members remain in
    the same role for the entirety of their careers.

2.  For one director-crew link, the edge weight calculation is invalid
    due to the fact that the director has not worked with anyone in the
    crew member's primary role and the crew member was working in a role
    other than their primary one while on the director's film. In this
    edge case, the edge weight is set equal to the collaboration score,
    which divides the number of collaborations by the total number of
    films.

3.  Due to the overwhelming amount of niche subroles listed on IMDB, the
    usage of the more general role category is more effective for
    network generation. Relying on subroles to get adequate model
    results puts the model at a disadvantage because naming conventions
    of different roles differ across productions and many subrole titles
    are tailor-made for the movie itself. For example, crew members who
    wrote the book that a movie is based on are part of the Writing
    Credits role, but have subroles that include the specific book or
    movie title in them.

# Visualization {#visualization .unnumbered}

## Process Overview {#process-overview .unnumbered}

Below is an overview of the steps I followed to complete the
visualization task. Within this task I contribute to the analysis
section by writing a plan for how to address each research question and
computing statistics in Gephi and NetworkX along the way. The
visualizations are integrated into the analysis section for each
research question.

1.  Wrote a plan for how to answer each research question using specific
    network analysis techniques and a list of visualizations.

2.  Identified the node and edge attributes required to achieve the
    visualizations to help with network generation.

3.  Created a toy network with the required attributes to practice
    visualization in Gephi.

4.  Planned additional visualizations like histograms, bar charts, and
    Sankey plots and described how each visualization helps to answer
    the research questions.

5.  Wrote code to create plots using the toy network.

6.  Applied code to finalized network generation and completed necessary
    troubleshooting.

7.  Visualized networks in Gephi according to the written plan below,
    taking snapshots of the relevant filtered and unfiltered data.

## Chosen Visualization Methods {#chosen-visualization-methods .unnumbered}

A list of the visualizations chosen to answer each research question.
After each question I provide reasoning for why I chose this
visualization and the question it may answer.

**Question 0: Characterize the director-crew network.**

-   Number of movies per director? Histogram of the number of movies for
    the subset of director nodes.

-   Frequency of each role? Bar chart showing the count of each role
    across the entire network.

-   Number of crews? Histogram showing the number of unique crew members
    for each director.

-   Degree distribution? Histogram showing the degree distribution.

**Question 1: Measure how roles of crew members fluctuate.**

-   Histogram showing the director influence distribution, calculated
    according to the described methods in the analysis section.

-   Career trajectory line plot case study: Pick a crew member and plot
    the number of movies they work on in each year to identify spikes
    after working with certain directors.

**Question 2: Measure how roles of crew members fluctuate.**

-   Network visualization: A network with the node size = number of
    unique roles (num_roles), node color = most frequent role (role),
    and edge size + edge color = constant. Take snapshots of the network
    filtered to different role types to answer the question: Which roles
    tend to work in other departments? Which crew member nodes have
    worked a uniquely large or small number of roles?

-   Sankey diagram: Visualize the flow of role transitions from start of
    career (start_role) to present (end_role). How do roles change over
    the course of a career? Are there any roles that frequently lead to
    other roles (ex. Writer to producer)?

-   Scatter plot of unique roles vs. total movies, colored by most
    frequent role: Do crew members tend to try new roles as they work on
    more movies, or maintain the same role?

-   Histogram of distribution of unique roles (num_roles) for all crew
    members: This visualization will show trends in industry overall.
    How many unique roles do crew members have over their career?

**Question 3: How widespread is the phenomenon of directors re-using the
same crew? Do renowned directors (and women/minority directors) tend to
work persistently with the same key collaborators compared to lesser
recognized directors or a random Hollywood director?**

-   Network visualizations: Node size = total number of films (to show
    how prolific the director is), node_color: based on demographic
    attributes (sex, race, queer), edge size = normalized collaboration
    score (derived in the calculation mentioned above), and edge color =
    constant. Take snapshots of the network filtered to different
    demographics to answer the question: How do the ego networks of
    directors of different demographics compare in terms of their
    collaboration score?

-   Histograms of collaborations normalized by movies for each
    demographic: For each key demographic, make a histogram of the
    collaboration scores. How common is it for directors of each
    demographic to work with the same crew?

-   Scatter plot of number of movies vs. unique crew members, colored by
    demographics: This scatter plot will let us identify trends and
    possibly communities within demographics (clusters in the scatter
    plot). As directors make more movies, do they tend to use the same
    crew or many different crews?

The chosen networks visualizations leverage network analysis concepts
like centrality, ego networks, and stylizing the network based on
important node/edge attributes to quantify role fluctuation and
collaboration patterns. Additional plots were selected to analyze career
trajectories, role changes, and potential differences in director
demographics.

# Analysis {#analysis .unnumbered}

## Overview of the Network {#overview-of-the-network .unnumbered}

There are 1415 featured movies in the network, 101 directors, and 4504
crew members.

The directors in the network each have directed anywhere from 3 to 53
films, with an average of 14.009 movies per director.

![A histogram showing the distribution of movies per
director.](0_histmoviesperdirector.png){#fig:movies_hist
width="0.33\\linewidth"}

There are a total of 8 crew roles, with frequencies as shown in Figure
8.

![A bar chart showing the frequencies of crew
roles.](0_crewmember_frequency.png){#fig:bar-chart
width="0.33\\linewidth"}

The total number of unique crew members in a director's network is
dependent on how many films they've directed, how large the crews were
for said films, and the director's personal tendencies when it comes to
rehiring the same crew members. All three of these aspects explain
different proportions of the number of unique crew members depending on
the director's specific circumstances.

![A histogram showing the distribution of the number of unique crew
members per director.](0_histuniquecrewperdirector.png){#fig:enter-label
width="0.33\\linewidth"}

![The degree distribution plot of the network (log
scale).](0_degreedistlog.png){#fig:log-label width="0.33\\linewidth"}

The degree distribution of the network closely resembles what we would
anticipate for a real-world network, with the plot following the
power-law distribution.

The network has an average shortest path length of 4.1956, an average
clustering coefficient of 0.0159, and a density of 0.00067.

## Question 1: Measure the influence of directors on crew members' careers. {#question-1-measure-the-influence-of-directors-on-crew-members-careers. .unnumbered}

In order to measure the overall influence of directors on the future
success of crew members, we calculate an influence metric. The
methodology of the metric calculation is as follows:

1.  For each crew member, we aggregate the total number of movies they
    worked on per year. Data is placed in a dictionary with years as the
    keys and the total numbers of movies that the crew member worked on
    during each time period as the values.

2.  Iterating through each year chronologically, we track the change in
    total number of films. If the crew member sees a 50% increase in the
    total number of films from one year to the next, we recognize the
    directors from the previous year as influential. (50% increase
    indicates that the total number of films from the recent time period
    is 1.5 times the size of the total number of films from the previous
    time period).

3.  Directors' influence scores are then increased by a factor of
    (1/number of directors from previous year)\*(number of movies in
    recent year/number of movies in previous year).

    1.  Dividing by the number of directors that the crew member worked
        with in their time period leading up to the jump in their
        success allows for the credit to be equally distributed across
        directors. Given the cumulative nature of this metric (influence
        is an overall score that is increased any time crew members
        experience significant growth immediately after working with a
        director), using a normalized value is more effective than a raw
        count (ex: adding 1 to each director's influence score each
        time).

    2.  Multiplying the 1/# of directors portion by the ratio of recent
        time period movies to former time period movies allows for the
        influence metric to be scaled by the significance of career
        growth. For example, a director's influence score will be
        increased by a larger amount if their former crew member sees
        500% growth in the total \# of movies they work on in a year
        than if the crew member only sees a 50% growth.

4.  Once all crew nodes have been accounted for, we create a table that
    contains director id numbers and their cumulative influence scores.

Here is a brief selection from the influence table calculated from our
network:

   **Director**   **Influence**
  -------------- ---------------
    nm0000142       53.759852
    nm0000229       53.053355
    nm0000165       43.418332
       \...           \...
    nm0001631       0.550000
    nm0036349       0.390244
    nm1883257       0.202128

The directors have influence scores ranging from 0.2021 to 53.7599, with
an average influence score of 11.435. This plot shows that the
distribution of influence is skewed to the right, as many directors have
influence scores of less than or equal to 15 and very few directors have
influence scores higher than 15.

![A histogram showing the distribution of director influence
values.](1_influence_hist.png){#fig:influence_hist
width="0.33\\linewidth"}

According to our calculations, the top ten most influential directors in
order are:

1.  Clint Eastwood (53.75985197273401)

2.  Steven Spielberg (53.05335525185566)

3.  Ron Howard (43.41833197931151)

4.  Ridley Scott (33.455846479716364)

5.  Tim Burton (31.72575494005538)

6.  Michael Bay (29.42764822060226)

7.  Robert Zemeckis (29.23757765500819)

8.  Martin Scorsese (28.077092366889264)

9.  Tyler Perry (27.29668109668107)

10. Steven Soderberg (26.923324044376663)

The directors with the lowest influence scores are Pablo Larrain, Andrea
Arnold, Sarah Polley, Lynn Shelton, and Charles Burnett. From a
qualitative standpoint, these rankings make sense - specifically the
most influential directors - as they are all quite well known and often
work on films that require lots of crew, so they have very extensive
networks and therefore a larger sample size of crew members who might
display career growth after working with them.

There are two primary drawbacks of this metric: the lack of
normalization by crew member role type and dataset limitations.

Certain role types allow for crew members to be involved in more movies
in the same time period than others. For example, makeup artists are
able to work on more movies in a year than producers are, as producers
are involved for the entire duration of the film-making process whereas
makeup artists are only involved during the filming portion of the
production. This is somewhat addressed by the usage of a threshold value
rather than an exact number - we look for specific percentage increases,
not point increases, which helps ensure credit is given to the crew
members who are generally able to do to fewer films in a year.

Because our dataset only contains well-known directors, it is likely
that many crew members have credits from the years prior to their first
time working with a director from our dataset as well as credits during
the same time frame that are not accounted for in the network. The
metric also fails if a crew member achieved significant career growth in
the first three years and plateaued or didn't see that high of a rate of
change again.

However, we conclude that this metric is a valid way of evaluating the
impact of directors on their crew members' career trajectories provided
that we consider the values within the context of the limitations.

### Case study: Daniel Lepervanche (id = nm4951849) {#case-study-daniel-lepervanche-id-nm4951849 .unnumbered}

Daniel Lepervanche is a crew member whose primary role is working in the
Sound Department. He experienced a substantial jump in his career
between 2015 and 2016.

![A line plot showing the career trajectory of a crew member, Daniel
Lepervanche.](1_career_traj.png){#fig:career_traj
width="0.5\\linewidth"}

In 2015, Lepervanche worked with Richard Linklater, who, according to
our measure of influence, played some part in enabling Lepervanche to
find significant success in the following year. Due to this instance,
Linklater's influence score was increased by (5 films in 2016/1 film in
2015) \* (1/1 director in 2015) = 5/1 \* 1 = 5.

## Question 2: Measure how the roles of crew members fluctuate. {#question-2-measure-how-the-roles-of-crew-members-fluctuate. .unnumbered}

The whole group discussed the metric for crew member role fluctuation.
Anna implemented the metrics and analyzed the visualizations.

**Results and Metrics**

We conclude that overall there is very little fluctuation in crew member
roles, with the most common changes occurring in the writing and
production departments. To answer this research question, we devised a
metric to quantify how and how much a crew member's role changed. We
derived the number of roles over dataset career as:

``` {#lst:copy .python language="Python" label="lst:copy"}
attrs[name]['num_roles'] = len(np.unique(attrs[name]['roles'].split(', ')))
```

where \['roles'\] is a list of all of the roles the crew member had on
each movie for which their name id was listed. We were also interested
to know which career changes were common, so we compared role
fluctuations between departments using visualizations.

**Statistics and Visualizations Reveal That Crew Members Change Roles
Infrequently**

The statistics and visualizations provide futher insights into how the
roles of crew members fluctuate throughout their careers in the film
industry. The two histograms (Figure 13, Figure 14) reveal that the vast
majority of crew members had only one role (97.4%) or one subrole
(79.8%). This trend indicates that there is very little role fluctuation
among the crews of the 101 directors selected.

  -----------------------------------------------------------------------
  **Attribute   **Object       **Definition**           **Sample
  name**        type**                                  attribute value**
  ------------- -------------- ------------------------ -----------------
  role          string         The primary role of the  'director'
                               node, 'director' for all 
                               director nodes           

  sex           string         Sex of the director      'F' for female,
                                                        'M' for male

  race          string         Racial identity of the   'B' for Black,
                               director                 'L' for Latino,
                                                        'A' for Asian,
                                                        'I' for
                                                        Indigenous, 'W'
                                                        for white

  labels (if    string         'H' if the director is   'H' for highest
  applicable)                  one of the top 20        grossing, 'Q' for
                               highest-grossing         LGBTQ+
                               directors (there are     
                               only 20 directors with   
                               the label 'H' in the     
                               original director file), 
                               'Q' if the director is   
                               LGBTQ+. Director nodes   
                               only possess this        
                               attribute if they belong 
                               to one or both of these  
                               categories.              

  total_films   integer        Number of total films    
                               directed or co-directed  
                               by the director (does    
                               not include films in     
                               which the director took  
                               on other roles like      
                               writer/producer under    
                               another director's       
                               direction).              

  collabs       string (use    List of all collaborator 'Writing Credits,
                .split.(', ')  roles for each movie     Makeup
                to convert to  (with repeats). For      Department,
                list)          example, if a film has 2 Makeup
                               writers, 1 producer, and Department, Sound
                               2 makeup artists, that   Department,
                               will appear in the       Special Effects
                               'collabs' attribute as   by, Writing
                               'Writing Credits,        Credits, Produced
                               Writing Credits,         by, Music by,
                               Produced by, Makeup      Makeup
                               department, Makeup       Department, Sound
                               department.' This is     Department, Sound
                               then done for all films  Department,
                               to keep track of how     Writing Credits,
                               common certain roles are Writing Credits,
                               and weight the edges     Produced by,
                               between crew members and Produced by,
                               directors proportionally Produced by,
                               to the                   Makeup
                               significance/rareness of Department, Sound
                               their role.              Department,
                                                        Special Effects
                                                        by, Special
                                                        Effects by,
                                                        Writing Credits,
                                                        Writing Credits'
  -----------------------------------------------------------------------

**Figure 2 - Crew node attributes**

  ---------------------------------------------------------------------------
  **Attribute       **Object     **Definition**               **Sample
  name**            type**                                    attribute
                                                              value**
  ----------------- ------------ ---------------------------- ---------------
  roles             String (use  List of all roles possessed  'Sound
                    .split.(',   by the crew member (with     Department,
                    ') to        repeats) over the course of  Sound
                    convert to   their career. Indexes        Department'
                    list)        correspond with the          
                                 'directors' and 'years'      
                                 attributes Ex: Crewmember    
                                 possessed the role of        
                                 roles\[0\] in the year       
                                 years\[0\] under the         
                                 director directors\[0\].     

  subroles          String (use  List of all subroles         're-recording
                    .split.(',   possessed by the crew member mixer,
                    ') to        (with repeats) over the      re-recording
                    convert to   course of their career. Some mixer'
                    list)        individuals possess multiple 
                                 subroles in the same film    
                                 (ex: 'hair department head'  
                                 and 'makeup department       
                                 head'), so this list is not  
                                 indexable in the same way as 
                                 the roles attribute.         

  directors         String (use  List of directors associated 'nm0001054,
                    .split.(',   with each film the crew      nm0001054'
                    ') to        member worked on (with       
                    convert to   repeats)                     
                    list)                                     

  years             String (use  List of years associated     '1987, 1984'
                    .split.(',   with each film the crew      
                    ') to        member worked on (with       
                    convert to   repeats if multiple films in 
                    list)        the same year)               

  total_films       Integer      Total number of films the    
                                 crew member has worked on    

  total_directors   Integer      Total number of unique       
                                 directors that the           
                                 crewmember worked with       

  min_year          String       The year of the first movie  '1984'
                                 the crew member worked on    

  max_year          String       The year of the most recent  '1987'
                                 movie the crew member worked 
                                 on                           

  start_role        String       The role the crew member     'Sound
                                 possessed in their first     Department'
                                 movie                        

  end_role          String       The role the crew member     'Sound
                                 possessed in their most      Department'
                                 recent movie                 

  num_roles         Integer      The total number of unique   
                                 roles taken on by the crew   
                                 member in their career       

  num_subroles      Integer      The total number of unique   
                                 subroles taken on by the     
                                 crew member in their career  

  role              String       Primary role possessed by    'Writing
                                 the crew member (most        Credits'
                                 frequent role in the course  
                                 of their career/in the       
                                 'roles' attribute list)      
  ---------------------------------------------------------------------------
