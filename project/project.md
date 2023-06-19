---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/project.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../lectures/css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

# Data Visualization Project Summary

{(whoami|}Valentin Madzharov valentin.madzharov@city.ac.uk{|whoami)}

{(task|}

You should complete this datavis project summary document and submit it, along with any necessary supplementary files to **Moodle** by **Sunday 18th December, 5pm UK time**. Submissions will be awarded up to **80 marks** towards your coursework assessment total.

You are also encouraged to regularly commit and push changes to your datavis project throughout the term as you develop your project.

{|task)}

{(questions|}

- How has the game of basketball developed in the NBA over time?

- How has Stephen Curry influenced the game of basketall and is his influence as substantial as described by the media?

- Is the three point shot worth too many points and if so why or why not?

{|questions)}

{(visualization|}

```elm {v interactive}
developmentMultiLineGraph : Spec
developmentMultiLineGraph =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" totalsTableSorted |> nums)
                << dataColumn "Stat" (numColumn "Stat" totalsTableSorted |> nums)
                << dataColumn "StatType" (strColumn "StatType" totalsTableSorted |> strs)

        enc =
            encoding
                << position X [ pName "Season", pTitle "Season", pAxis [ axLabelAngle 0, axGrid False, axTickColor "black", axLabelFontSize 12.0, axTitleFont "Times New Roman", axLabelColor "black", axLabelPadding 12, axTitleFontSize 20.0, axTitleColor "black", axDomainColor "black", axTitlePadding 40.0 ] ]

        transSelFilter =
            transform
                << filter (fiSelectionEmpty "hover")

        lineColors =
            categoricalDomainMap [ ( "Points", "rgb(0, 191, 255)" ), ( "Field Goals Attempted", "rgb(255, 0, 0)" ), ( "Rebounds", "rgb(rgb(0, 128, 0))" ), ( "Field Goals Made", "rgb(255, 255, 0)" ), ( "Assists", "rgb(255, 0, 102)" ), ( "Personal Fouls", "rgb(0, 51, 102)" ), ( "Turnovers", "rgb(153, 0, 0)" ), ( "Steals", "rgb(102, 255, 51)" ), ( "Blocks", "rgb(204, 0, 153)" ) ]

        enc1 =
            encoding
                << position Y [ pName "Stat", pTitle "Statistics Totals Per Season ", pQuant, pAxis [ axLabelAngle 0, axGrid False, axTickColor "black", axGridColor "black", axLabelFontSize 12.0, axLabelPadding 12, axLabelColor "black", axTitleFontSize 20.0, axTitleFont "Times New Roman", axTitleColor "black", axDomainColor "black", axTitlePadding 40.0 ] ]
                << color [ mName "StatType", mSort [ soByField "Stat" opMax, soDescending ], mTitle "Stats", mScale lineColors, mLegend [ leTitleFontSize 20.0, leStrokeColor "black", leLabelFontSize 15, leTitleFont "Times New Roman", leLabelFont "Times New Roman", lePadding 12 ] ]

        spec1 =
            asSpec
                [ enc1 []
                , layer
                    [ asSpec [ line [] ]
                    , asSpec [ transSelFilter [], point [] ]
                    ]
                ]

        ps =
            params
                << param "hover"
                    [ paSelect sePoint
                        [ seFields [ "Season" ]
                        , seOn "mouseover"
                        , seClear "mouseout"
                        , seNearest True
                        ]
                    ]

        transPivot =
            transform
                << pivot "StatType" "Stat" [ piGroupBy [ "Season" ] ]

        enc2 =
            encoding
                << opacity [ mCondition (prParamEmpty "hover") [ mNum 0.3 ] [ mNum 0 ] ]
                << tooltips
                    [ [ tName "Points", tQuant, tTitle "Points" ]
                    , [ tName "Field Goals Attempted", tQuant, tTitle "Field Goals Attempted" ]
                    , [ tName "Rebounds", tQuant, tTitle "Rebounds" ]
                    , [ tName "Field Goals Made", tQuant, tTitle "Field Goals Made" ]
                    , [ tName "Assists", tQuant, tTitle "Assists" ]
                    , [ tName "Personal Fouls", tQuant, tTitle "Personal Fouls" ]
                    , [ tName "Turnovers", tQuant, tTitle "Turnovers" ]
                    , [ tName "Steals", tQuant, tTitle "Steals" ]
                    , [ tName "Blocks", tQuant, tTitle "Blocks" ]
                    , [ tName "Season", tQuant, tTitle "Season" ]
                    ]

        spec2 =
            asSpec [ ps [], transPivot [], enc2 [], rule [] ]

        annotationData =
            dataFromRows []
                << dataRow
                    [ ( "Season", str "1999" )
                    , ( "annotation", str "1998–99 NBA lockout season resulting in a late season start (50 total games instead of 82)." )
                    , ( "Stat", num 250000 )
                    ]
                << dataRow
                    [ ( "Season", str "2012" )
                    , ( "annotation", str "2011-2012 NBA lockout season resulting in a late season start (66 total games instead of 82)." )
                    , ( "Stat", num 140000 )
                    ]
                << dataRow
                    [ ( "Season", str "2019" )
                    , ( "annotation", str "COVID Seasons (72 Total Games)." )
                    , ( "Stat", num 170000 )
                    ]
                     << dataRow
                    [ ( "Season", str "2009" )
                    , ( "annotation", str "Stephen Curry Gets Drafted.(2009)" )
                    , ( "Stat", num 2000 )
                    ]

        encAnnotation =
            encoding
                << position X [ pName "Season" ]
                << position Y [ pName "Stat", pQuant ]
                << text [ tName "annotation" ]

        specAnnotation =
            asSpec [ annotationData [], encAnnotation [], textMark [ maDy -5 ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coBackground "Orange")
                << configuration (coTitle [ ticoFont "Times New Roman", ticoFontSize 40, ticoAnchor anMiddle, ticoColor "Black" ])
                << configuration (coPadding (paSize 60))
    in
    toVegaLite [ title "NBA Statistics as Totals Per Season" [], width 1370, height 500, data [], enc [], cfg [], layer [ spec1, spec2, specAnnotation ] ]
```

```elm {v}
salaryLinePointGraph : Spec
salaryLinePointGraph =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" salaryTable |> nums)
                << dataColumn "AvarageYearlySalaryCap" (numColumn "AvarageYearlySalaryCap" salaryTable |> nums)

        enc =
            encoding
                << position X [ pName "Season", pAxis [ axLabelAngle 0, axGrid False, axTickColor "white", axGridColor "white", axLabelColor "white", axLabelPadding 12, axLabelFontSize 12.0, axTitleFontSize 20.0, axTitleFont "Times New Roman", axTitleColor "white", axTitlePadding 60.0, axDomainColor "white" ], pTitle "Season" ]
                << position Y [ pName "AvarageYearlySalaryCap", pQuant, pAxis [ axLabelAngle 0, axGrid True, axTickColor "white", axGridColor "white", axLabelColor "white", axLabelFontSize 12.0, axLabelPadding 12, axTitleFontSize 20.0, axTitleFont "Times New Roman", axTitleColor "white", axDomainColor "white" ], pTitle "Avarage Yearly Salary Cap in 2022 Dollars considering Inflation" ]

        cfg =
            configure
                << configuration (coBackground "green")
                << configuration (coTitle [ ticoFont "Times New Roman", ticoFontSize 40, ticoColor "White" ])
                << configuration (coScale [ sacoBandPaddingInner 12.1 ])
                << configuration (coAxis [ axcoGridOpacity 0.1, axcoTicks False ])
                << configuration (coPadding (paSize 30))
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxisDiscrete [ axcoMaxExtent 1 ])
    in
    toVegaLite [ title "Review of teams Salary cap since the 80's" [], width 1550, height 500, data [], rule [], cfg [], enc [], line [ maPoint (pmMarker [ maFill "white" ]), maFilled True, maSize 10, maOpacity 0.5, maColor "white" ] ]
```

```elm {v}
positionBarGraph : Spec
positionBarGraph =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" threePointPositionData |> nums)
                << dataColumn "Stat" (numColumn "Stat" threePointPositionData |> nums)
                << dataColumn "Position" (strColumn "Position" threePointPositionData |> strs)

        backgroundEnc =
            encoding
                << position X [ pName "Season", pAxis [ axLabelAngle 0, axGrid False, axTickColor "white", axGridColor "white", axLabelColor "white", axLabelPadding 12, axLabelFontSize 12.0, axTitleFontSize 20.0, axTitleFont "Times New Roman", axTitleColor "white", axTitlePadding 60.0, axDomainColor "white" ], pTitle "Season" ]
                << position Y [ pName "Stat", pAggregate opSum, pTitle "Stat", pAxis [ axLabelAngle 0, axGrid True, axTickColor "white", axGridColor "white", axLabelColor "white", axLabelPadding 12, axLabelFontSize 12.0, axTitleFontSize 20.0, axTitleFont "Times New Roman", axTitleColor "white", axTitlePadding 60.0, axDomainColor "white" ], pTitle "Number of 3 pointers made per position since 2000" ]

        backgroundSpec =
            asSpec [ backgroundEnc [], bar [ maColor "black" ] ]

        groupedEnc =
            encoding
                << position X [ pName "Season" ]
                << position XOffset [ pName "Position" ]
                << position Y [ pName "Stat", pAggregate opSum ]
                << color [ mName "Position", mLegend [ leLabelLimit 170, leTitleFontSize 20.0, leStrokeColor "white", leLabelFontSize 15, leLabelColor "white", leTitleFont "Times New Roman", leTitleColor "white", leLabelFont "Times New Roman", lePadding 15 ] ]

        cfg =
            configure
                << configuration (coBackground "black")
                << configuration (coTitle [ ticoFont "Times New Roman", ticoFontSize 40, ticoColor "White" ])
                << configuration (coScale [ sacoBandPaddingInner 0.1 ])
                << configuration (coAxis [ axcoGridOpacity 0.1, axcoTicks False ])
                << configuration (coPadding (paSize 60))
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxisDiscrete [ axcoMaxExtent 0 ])
    in
    toVegaLite [ title "Three Point Shots attempted by position since 2000 " [], data [], width 1275, height 500, groupedEnc [], bar [], cfg [], layer [ backgroundSpec ] ]
```

```elm {v}
threePointDataLineFacetGraph : Spec
threePointDataLineFacetGraph =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" threePointStats |> nums)
                << dataColumn "Stat" (numColumn "Stat" threePointStats |> nums)
                << dataColumn "StatType" (strColumn "StatType" threePointStats |> strs)

        enc =
            encoding
                << position X [ pName "Season", pAxis [ axLabelAngle 0, axGrid False, axTickColor "Blue", axGridColor "Blue", axLabelColor "Blue", axLabelFontSize 12.0, axLabelPadding 12, axTitleFontSize 20.0, axTitleFont "Times New Roman", axTitleColor "Blue", axDomainColor "Blue" ], pTitle "" ]
                << position Y [ pName "Stat", pQuant, pAxis [ axLabelAngle 0, axGrid True, axTickColor "Blue", axGridColor "Blue", axLabelColor "Blue", axLabelFontSize 12.0, axLabelPadding 12, axTitleFontSize 20.0, axTitleFont "Times New Roman", axTitleColor "Blue", axDomainColor "Blue" ], pTitle "" ]
                << row [ fName "StatType", fHeader [ hdTitle "Stat", hdTitleFontSize 20, hdLabelAngle 0, hdLabelAnchor anMiddle, hdLabelAngle 270, hdLabelFontSize 15, hdLabelColor "blue", hdTitleColor "blue" ] ]

        res =
            resolve
                << resolution (reScale [ ( chY, reIndependent ) ])

        cfg =
            configure
                << configuration (coBackground "yellow")
                << configuration (coTitle [ ticoFont "Times New Roman", ticoFontSize 40, ticoColor "Blue", ticoAnchor anMiddle ])
                << configuration (coScale [ sacoBandPaddingInner 12.1 ])
                << configuration (coAxis [ axcoGridOpacity 0.1, axcoTicks True ])
                << configuration (coPadding (paSize 60))
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxisDiscrete [ axcoMaxExtent 1 ])
    in
    toVegaLite [ title "Three Point Shot Evaluation " [], width 1500, height 240, cfg [], data [], res [], enc [], line [] ]
```

```elm {v}
shapesGraph : Spec
shapesGraph =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" mvpTable |> nums)
                << dataColumn "MVP" (strColumn "MVP" mvpTable |> strs)
                << dataColumn "POS" (strColumn "POS" mvpTable |> strs)
        shapeColors=
            categoricalDomainMap [ ( "Point Guard", "gold" ), ( "Shooting Guard", "gold" ), ( "Small Forward", "gold" ), ( "Power Forward", "gold" ), ( "Center", "gold" ) ]
        enc =
            encoding
                << position X [ pName "Season",pAxis [ axLabelAngle 0, axGrid False, axTickColor "gold", axGridColor "gold", axLabelColor "gold", axLabelFontSize 12.0, axLabelPadding 12, axTitleFontSize 20.0, axTitleFont "Times New Roman", axTitleColor "gold", axDomainColor "gold" ], pTitle ""  ]
                << color [ mName "POS",mTitle"",mScale shapeColors,mLegend [ leLabelLimit 170, leTitleFontSize 20.0, leLabelFontSize 15, leLabelColor "Black", leTitleFont "Times New Roman",leSymbolFillColor "Black", leTitleColor "Black", leLabelFont "Times New Roman", lePadding 15 ]]
                << shape [ mName "POS",mTitle"", mLegend [ leLabelLimit 170, leTitleFontSize 20.0, leLabelFontSize 15, leLabelColor "gold", leTitleFont "Times New Roman",leSymbolFillColor "Gold", leTitleColor "gold", leLabelFont "Times New Roman", lePadding 15 ]]
                
        cfg =
            configure
                << configuration (coBackground "Black")
                << configuration (coTitle [ ticoFont "Times New Roman", ticoFontSize 40, ticoColor "gold", ticoAnchor anMiddle ])
                << configuration (coScale [ sacoBandPaddingInner 12.1 ])
                << configuration (coAxis [ axcoGridOpacity 0.1, axcoTicks False ])
                << configuration (coPadding (paSize 60))
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxisDiscrete [ axcoMaxExtent 1 ])
    in
    toVegaLite
        [ title"The MVPs"[],width 1400
        , height 200
        , data []
        , enc []
        ,cfg[] 
        , point [ maFilled True, maSize 200]
        ]
```

{|visualization)}

The Data


As a relatively new fan of the NBA(started regularly watching the sport of basketball and the NBA two seasons ago), I wanted to see the value of data visualisation in sports. This is  because of the fact of how significant statistics have become in sports and data sports science. I was motivated by this and it made me curious to see what kind of new trends elements and facts I will be able to discover by developing data visualisations and also by then using the data visualisation and analysing them in order to see if I can answer my questions. 

This project relies on two different datasets which are both extracted from the National Basketball Association(NBA) in the time period between the 1980 season and the 2022 season. The data consists of game statistics which professionals use to create verdicts on player, team and season performances. Because of that, I wanted to focus on the game of basketball and its progress per annum I narrowed down the data by only using the seasonal Totals data which allowed me to work over a dataset which wasn’t too broad but also not too narrow.

The data extracted were stored in a CSV format within my submission document which allowed me to manipulate the structure of the data in order to fit the visualisation structure. The data was already extracted and formatted in a tidy manner which allowed me to have data shaped into being suitable for encoding with channels which were important to me as it allowed me to use a number of sources of data for my visualisations. Even tho I had two main sources of data when the data was extracted it was done by only extracting specific fields useful per visualisation which made each of my tables Tidy and easy to understand and edit if needed.




{(insights|}

1.
  How has the game of basketball developed in the NBA over time? (Insight one)

I will begin answering this question by having an analytical overview of the “NBA Statistics as Totals Per Season”. 
To begin with, I wasn't expecting incredible slopes in the 1998-99 and 2011-12 regions which led me to carry out some research on why these anomalies took place. Before even carrying out the research, however, I already understood how valuable data visualisation really is because spotting this trend when looking with the naked eye at a large data set will be extremely difficult. In this insight, I will be writing about the different trends I was able to spot which allowed me to learn new information about the sport of basketball and how it has developed. 

The Spikes and Slopes

After looking up the season and what was so unique about them I found that both seasons had fewer games take place than the usual 82. This occurred due to the fact the General Managers of the teams in the NBA and the league executives were disagree on the way the superstar players like Patrick Ewing and previously Larry Bird were paid an incredible amount of money compared to the lowest-paid players. The League executives wanted to put a more firm salary cap on teams meaning that the superstars can still get paid an incredible amount however the lowest-paid player’s salaries were going to have had their minimum salaries raised adequately.

The game

Continuing to analyse the graph as a whole I compared the first and latest season on the graph and its values I discover that all statistics about the game have a had increased, however, the statistic which focused a lot more on the offensive side of the game had increased a lot more than the defensive statistics. An example of this is seeing how the number of points per season went from 197239 Points in a season to 272115 points in a season which shows that the number of points scored in the NBA increased by nearly a third, whereas when comparing the number of blocks went from 9561 to only 11594 which still shows an increase but is minimal compared to the number of points scored in a season. This amazed me as discovering the fact that the game has developed so much more on the offensive side in contrast to the defensive really put into perspective how the game has focused a lot more on the entertainment fast outsourcing model compared to the defensive graft-like model. This progression does actually makes sense because the more entertaining the game of basketball the more the fans worldwide and in the arenas which will skyrocket the revenue of the league which would also increase the player's salary and team salary cap which we can see in the Review of teams Salary cap’s since the 80’s where we can see the average increase of the team's salary caps has increased significantly from around the 9,000,000 all the way up to around 120,000,000. The final trend I was able to come across was the fact that is that the game of basketball is constantly evolving and its players are becoming more skilful with their scoring and they keep on adapting their game in order to stay relevant with the game and time. This is why we see an overall increase of all stats even if they don't increase on the same line of trajectory.








----------------------------------------------------------------------------------------------------------------------------------------
2.
  How has Stephen Curry influenced the game of basketball and is his influence as substantial as described by the media? (Insight two)

To answer the following question I will be considering Stephen Curry's skill set which is his outrageous 3-point shooting ability and efficiency rate. Stephen curry is the best shooter of all time undisputedly and the media has portrayed his impact on the game of basketball as ever-lasting this is because he showed the world of basketball how a player can consistently score three-point shots efficiently at a high volume. 

Using my visualisations I wanted to see if his impact on the game is as dramatic as I thought it would be and what trends have occurred since he entered the league. Stephen curry entered the league in 2009 and we can see that in our first visualisation.
When we, however, consider the “Three-point shots attempted since 2000” graph as a whole I was able to locate that up to 2013 excluding the lockout season there is a slight but steady increase in the number of 3-point shots attempted by all positions. However, from 2014 onwards we can see a serious increase up until the 2022 season. This increase in 3-point shots attempted is oddly matching with when Curry became an established household name in the game of basketball and when he won his first championship with the Golden State Warriors(2014).
 
The most surprising impact is the fact that a point guard (playmaker and shooter extremely skilful ) influenced the game of the centre position  (Physical big men with limited skill sets) by changing the way they play the game of basketball and forcing them to apply three-point shooting to their skillset which was a rarity in previous years. Comparing how many three-pointers all centres shot in the year 2000 to how many they did in 2022 is staggering as we see an increase from about 500 3 point shots attempted by centres in 2000 to around 5000 in 2022. His impact on the game is undeniable on the skilful players which are the point guards and shooting guards but this visualisation shows us that he has impacted the game in all positions which shows how severe his impact is on the game of basketball and all of its positions. 

----------------------------------------------------------------------------------------------------------------------------------------
3. 
Is the three point shot worth too many points and if so why or why not? (Insight three)
To answer this question I wanted to be able to see how the 3-point shot developed over time and does it contribute to field goals in general in an obscene manner.

The three-point shot is 3 points for a reason and that is it's further away from the basket so it's harder to score. Or is it?

When analysing my “Three-point shot Evaluation '' graph I first wanted to see if players are becoming more efficient the more 3s they shoot because that is what I assumed as an average fan because practice makes perfect in theory. Right?

Well, I came to the conclusion that basketball is not really theory as I came across the fact that players have stayed at the same efficiency of shooting 3s since 1995 which I found odd because I could clearly see that the number of 3s made and attempted increased however their efficiency stayed the same.  I imagine this is because even the teams that can shoot 3s well have to deal with tougher perimeter defence which doesn't allow them to take easy threes and win games in blowouts but in fact, they would have to settle for so two-point shots rather than taking contested 3s because they will be seen as inefficient and lead to inefficient conversion rate.

This leads me to believe to the fact that the number of threes taken wouldn't really matter because the players are not becoming more efficient which proves that the 3-point shot is not worth too many points but in fact, it is just right as you can see that the regular field goals taken on average are more efficiently converted than 3s as 3s are converted at 35% whereas regular field goals including 2s are converted at 45%.

{|insights)}

{(designJustification|}

1. Colours


“NBA Statistics as Totals Per Season”  Graph

The colours were specifically picked out for this graph. The X and Y axis and their labels ticks and titles were all made black and the background was selected to be orange in order the representative colour of the basketball to reference the game of basketball as a whole also all lines were hand-picked as I had an issue picking a scheme with colours which didn't clash with each other and the background. 

The hand-picked line colour scheme was intended to have a lot of either really bright like lime green or really dark like dark purple to make them very distinguishable for the user when they are trying to identify values and trends on the graph. 

“Three Point Shot Evaluation” Graph

The colours for this graph again had meaning behind its focal point which was Stephen Curry and the team he plays for which is the Golden State Warriors. The colour of jerseys of Stephen is blue and yellow which is symbolised by the 2 axes and the background colour including all titles and labels. 

“Review of teams Salary cap since the 80’s “ Graph

The colours of this graph symbolised the American dollar with the green and white colour scheme and this colour scheme stays consistent throughout the graph even within the area marked on the graph with the lighter white opacity.

Opacity

Throughout my visualisations, I used opacity either for the axis grid or when marking a specific area like in the salary graph. I used it in order to allow the users still distinguish data while keeping the elegance and consistency on the color choices and their brightness and vibrance.

----------------------------------------------------------------------------------------------------------------------------------------

2. Graph Selection

Development Multi Line Graph

I chose the “NBA Statistics as Totals Per Season”  Graph to be a multi-line interactive graph because the goal of the graph was to allow the user to find out any statistic on the timeline at ease and that's also why aligned the legend and hovering table to place all values in a descending order having the highest value being the line on the top mirroring the legend throughout the diagram.
Additionally, I believe that this  was the neatest possible way of displaying data individually per position without excessive overlapping as I came across when attempting to apply the data using a bar graph.

Salary Line Point  Graph

The Salary cap graph was pretty easy to choose I have seen real-life examples of how stock data is displayed efficiently and kept as user-friendly as possible. I Additionally added the marker which I filled in order to showcase how the salary cap as [eaked and sloped in comparison to the straight average line between the first and last value on the graph.

Position Bar Graph
With the position Bar graph, I considered how I can show a lot of yearly values neatly while making the visualisation easily readable by users.
This type of graph is also called a Nested bar Graph however I removed the total data combine per season and focused more on how individual categories/positions have developed 

Three-Point Facet Graph
I decide to use a facet graph as the goal of this visualisation was to be able to display independent values over the same X axis while having different Y axis. This way I was able to contrast the different statistics on the three-point shot while still being able to compare them closely together.

----------------------------------------------------------------------------------------------------------------------------------------

3.  Layout

The Layout was something I considered when creating my graphs and I wanted to stay consistent with it.

When developing my bar, multi-line and Shape graph I wanted to have the legend placed in the same location to keep the design consistent and clean as I believe that provides a good user experience because they will find it easier to use all graphs if they are designed in a similar manner and if they have the same design tendencies.

Also, the way the axis and the way the graphs were placed generally I made sure were consistent in order to ensure a clean design by making sure there isn't a lot of confusion about the way the graphs work but in fact, reinforce the traditional graphs and their designs which most people come across.

I also ensured to  remove any junk inc like axis ticks or axis grids which don't contribute to the user experience to keep the elegance of graphs and their consistent elements


{|designJustification)}

Sketches
![image](./images/IMG_1023.png)
![image](./images/IMG_1024.png)
![image](./images/IMG_1025.png)

{(references|}

Wood, J. (2015) Visualizing Personal Progress in Participatory Sports Cycling Events. IEEE Computer Graphics and Applications, 35(4), pp. 73-81.

Tufte, E. R. (2001) The visual display of quantitative information. Cheshire, Conn, Graphics Press.

VegaLite Gallery (2022) https://vega.github.io/vega-lite/examples/

Kirk, A. (2019) Chapter 9: Colour, pp.249-276 in Data Visualisation: A Handbook for Data Driven Design, 2nd Edition, Sage.

Gleicher, M., Albers, D., Walker, R., Jusufi, I., Hansen, C. and Roberts, J. (2011) Visual comparison for information visualization. Information Visualization 10(4), pp.289-309.

Wickham, H. (2014) Tidy Data, Journal of Statistical Software, 59(10).

Tidyverse (2019) R Packages for Data Science (Some additional context for tidy data, in this case using the R statistics package)

{|references)}

```elm {l=hidden}
mvpTable : Table
mvpTable =
    """

Season,MVP,POS,

2022, N. Jokić,Center,
2021, N. Jokić,Center,
2020, G. Antetokounmpo,Center,
2019, G. Antetokounmpo,Center,
2018, J. Harden,Shooting Guard,
2017, R. Westbrook,Point Guard,
2016, S. Curry,Point Guard,
2015, S. Curry,Point Guard,
2014, K. Durant,Small Forward,
2013, L. James,Small Forward,
2012, L. James,Small Forward,
2011, D. Rose,Point Guard,
2010, L. James,Small Forward,
2009, L. James,Small Forward,
2008, K. Bryant,Shooting Guard,
2007, D. Nowitzki,Power Forward,
2006, S. Nash,Point Guard,
2005, S. Nash,Point Guard,
2004, K. Garnett,Power Forward
2003, T. Duncan,Power Forward,
2002, T. Duncan,Power Forward,
2001, A. Iverson,Point Guard,
2000, S. O'Neal,Center,
1999, K. Malone,Power Forward,
1998, M. Jordan,Shooting Guard,
1997, K. Malone,Power Forward,
1996, M. Jordan,Shooting Guard,
1995, D. Robinson,Power Forward,
1994, H. Olajuwon,Center,
1993, C. Barkley,Power Forward,
1992, M. Jordan,Shooting Guard,
1991, M. Jordan,Shooting Guard,
1990, M. Johnson,Point Guard,
1989, M. Johnson,Point Guard,
1988, M. Jordan,Shooting Guard,
1987, M. Johnson,Point Guard,
1986, L. Bird,Power Forward,
1985, L. Bird,Power Forward,
1984, L. Bird,Power Forward,
1983, M. Malone,Power Forward,
1982, M. Malone,Power Forward,
1981, J. Erving,Small Forward,
1980, K. Abdul-Jabbar,Center,

"""
        |> fromCSV
````
```elm {l=hidden}
totalsTableSorted : Table
totalsTableSorted =
    """

StatType,Season,Stat,

Assists,2022,60636,
Assists,2021,53577 ,
Assists,2020,51649 ,
Assists,2019,60483 ,
Assists,2018,57163 ,
Assists,2017,55660 ,
Assists,2016,54832 ,
Assists,2015,54202 ,
Assists,2014,54131 ,
Assists,2013,54415 ,
Assists,2012,41533 ,
Assists,2011,52887 ,
Assists,2010,52258 ,
Assists,2009,51605 ,
Assists,2008,53506 ,
Assists,2007,52365 ,
Assists,2006,50711 ,
Assists,2005,52342 ,
Assists,2004,50658 ,
Assists,2003,51133 ,
Assists,2002,52131 ,
Assists,2001,51786 ,
Assists,2000,53130 ,
Assists,1999,30050 ,
Assists,1998,52386 ,
Assists,1997,52426 ,
Assists,1996,53950 ,
Assists,1995,51729 ,
Assists,1994,54003 ,
Assists,1993,54705 ,
Assists,1992,54192 ,
Assists,1991,54782 ,
Assists,1990,55023 ,
Assists,1989,52431 ,
Assists,1988,48599 ,
Assists,1987,48998 ,
Assists,1986,49065 ,
Assists,1985,49515 ,
Assists,1984,49397 ,
Assists,1983,48862 ,
Assists,1982,47438 ,
Assists,1981,48034 ,
Assists,1980,46567 ,
Blocks,2022,11594 ,
Blocks,2021,10525 ,
Blocks,2020,10378 ,
Blocks,2019,12185 ,
Blocks,2018,11846 ,
Blocks,2017,11669 ,
Blocks,2016,12193 ,
Blocks,2015,11797 ,
Blocks,2014,11594 ,
Blocks,2013,12622 ,
Blocks,2012,10086 ,
Blocks,2011,11966 ,
Blocks,2010,11947 ,
Blocks,2009,11817 ,
Blocks,2008,11651 ,
Blocks,2007,11329 ,
Blocks,2006,11560 ,
Blocks,2005,11963 ,
Blocks,2004,12022 ,
Blocks,2003,11928 ,
Blocks,2002,12422 ,
Blocks,2001,12505 ,
Blocks,2000,12293 ,
Blocks,1999,7196 ,
Blocks,1998,12062 ,
Blocks,1997,11674 ,
Blocks,1996,12038 ,
Blocks,1995,11430 ,
Blocks,1994,11579 ,
Blocks,1993,11558 ,
Blocks,1992,12216 ,
Blocks,1991,11624 ,
Blocks,1990,11207 ,
Blocks,1989,10949 ,
Blocks,1988,10160 ,
Blocks,1987,10411 ,
Blocks,1986,9910 ,
Blocks,1985,10019 ,
Blocks,1984,10004 ,
Blocks,1983,10566 ,
Blocks,1982,10131 ,
Blocks,1981,10036 ,
Blocks,1980,9561 ,
Personal Fouls,2022,48306 ,
Personal Fouls,2021,41669 ,
Personal Fouls,2020,44004 ,
Personal Fouls,2019,51425 ,
Personal Fouls,2018,48837 ,
Personal Fouls,2017,48950 ,
Personal Fouls,2016,49854 ,
Personal Fouls,2015,49728 ,
Personal Fouls,2014,50923 ,
Personal Fouls,2013,48775 ,
Personal Fouls,2012,38744 ,
Personal Fouls,2011,50953 ,
Personal Fouls,2010,51309 ,
Personal Fouls,2009,51765 ,
Personal Fouls,2008,51709 ,
Personal Fouls,2007,54666 ,
Personal Fouls,2006,55986 ,
Personal Fouls,2005,55671 ,
Personal Fouls,2004,51006 ,
Personal Fouls,2003,51730 ,
Personal Fouls,2002,50497 ,
Personal Fouls,2001,53143 ,
Personal Fouls,2000,55408 ,
Personal Fouls,1999,32206 ,
Personal Fouls,1998,53278 ,
Personal Fouls,1997,52602 ,
Personal Fouls,1996,54806 ,
Personal Fouls,1995,51961 ,
Personal Fouls,1994,49075 ,
Personal Fouls,1993,51269 ,
Personal Fouls,1992,49227 ,
Personal Fouls,1991,51304 ,
Personal Fouls,1990,51478 ,
Personal Fouls,1989,48504 ,
Personal Fouls,1988,45461 ,
Personal Fouls,1987,46277 ,
Personal Fouls,1986,47543 ,
Personal Fouls,1985,47036 ,
Personal Fouls,1984,48573 ,
Personal Fouls,1983,48373 ,
Personal Fouls,1982,49352 ,
Personal Fouls,1981,47265 ,
Personal Fouls,1980,43952 ,
Points,2022,272115 ,
Points,2021,242117 ,
Points,2020,236788 ,
Points,2019,273573 ,
Points,2018,261580 ,
Points,2017,259753 ,
Points,2016,252572 ,
Points,2015,246035 ,
Points,2014,248482 ,
Points,2013,241223 ,
Points,2012,190594 ,
Points,2011,244894 ,
Points,2010,247100 ,
Points,2009,245879 ,
Points,2008,245811 ,
Points,2007,242899 ,
Points,2006,238641 ,
Points,2005,239109 ,
Points,2004,222097 ,
Points,2003,226102 ,
Points,2002,227043 ,
Points,2001,225459 ,
Points,2000,231789 ,
Points,1999,132792 ,
Points,1998,227269 ,
Points,1997,230433 ,
Points,1996,236619 ,
Points,1995,224518 ,
Points,1994,224737 ,
Points,1993,233073 ,
Points,1992,233155 ,
Points,1991,235370 ,
Points,1990,236882 ,
Points,1989,223797 ,
Points,1988,203993 ,
Points,1987,207338 ,
Points,1986,207875 ,
Points,1985,209041 ,
Points,1984,207668 ,
Points,1983,204658 ,
Points,1982,204776 ,
Points,1981,203886 ,
Points,1980,197239 ,
Steals,2022,18772 ,
Steals,2021,16356 ,
Steals,2020,16200 ,
Steals,2019,18779 ,
Steals,2018,18983 ,
Steals,2017,18950 ,
Steals,2016,19303 ,
Steals,2015,19031 ,
Steals,2014,18895 ,
Steals,2013,19178 ,
Steals,2012,15200 ,
Steals,2011,18023 ,
Steals,2010,17759 ,
Steals,2009,17883 ,
Steals,2008,17897 ,
Steals,2007,17807 ,
Steals,2006,17632 ,
Steals,2005,18489 ,
Steals,2004,18864 ,
Steals,2003,18884 ,
Steals,2002,18538 ,
Steals,2001,18600 ,
Steals,2000,18868 ,
Steals,1999,12113 ,
Steals,1998,19943 ,
Steals,1997,19486 ,
Steals,1996,18952 ,
Steals,1995,18330 ,
Steals,1994,19652 ,
Steals,1993,18932 ,
Steals,1992,19131 ,
Steals,1991,19014 ,
Steals,1990,18881 ,
Steals,1989,18619 ,
Steals,1988,16055 ,
Steals,1987,16264 ,
Steals,1986,16576 ,
Steals,1985,16122 ,
Steals,1984,16041 ,
Steals,1983,16778 ,
Steals,1982,16107 ,
Steals,1981,16964 ,
Steals,1980,16993 ,
Turnovers,2022,33856 ,
Turnovers,2021,29887 ,
Turnovers,2020,30802 ,
Turnovers,2019,34644 ,
Turnovers,2018,35092 ,
Turnovers,2017,34320 ,
Turnovers,2016,35384 ,
Turnovers,2015,35312 ,
Turnovers,2014,36034 ,
Turnovers,2013,35767 ,
Turnovers,2012,28860 ,
Turnovers,2011,35065 ,
Turnovers,2010,34982 ,
Turnovers,2009,34519 ,
Turnovers,2008,34719 ,
Turnovers,2007,37232 ,
Turnovers,2006,35459 ,
Turnovers,2005,35677 ,
Turnovers,2004,35610 ,
Turnovers,2003,35471 ,
Turnovers,2002,34389 ,
Turnovers,2001,35783 ,
Turnovers,2000,36788 ,
Turnovers,1999,22221 ,
Turnovers,1998,36823 ,
Turnovers,1997,37252 ,
Turnovers,1996,37641 ,
Turnovers,1995,35303 ,
Turnovers,1994,35424 ,
Turnovers,1993,35230 ,
Turnovers,1992,34430 ,
Turnovers,1991,35515 ,
Turnovers,1990,35555 ,
Turnovers,1989,35312 ,
Turnovers,1988,31563 ,
Turnovers,1987,32040 ,
Turnovers,1986,33645 ,
Turnovers,1985,33694 ,
Turnovers,1984,33755 ,
Turnovers,1983,36038 ,
Turnovers,1982,33435 ,
Turnovers,1981,35354 ,
Turnovers,1980,34166 ,
Rebounds,2022,109347 ,
Rebounds,2021,95686 ,
Rebounds,2020,94957 ,
Rebounds,2019,111107 ,
Rebounds,2018,107052 ,
Rebounds,2017,107046 ,
Rebounds,2016,107645 ,
Rebounds,2015,106504 ,
Rebounds,2014,105161 ,
Rebounds,2013,103575 ,
Rebounds,2012,83513 ,
Rebounds,2011,101816 ,
Rebounds,2010,102640 ,
Rebounds,2009,101586 ,
Rebounds,2008,103271 ,
Rebounds,2007,100994 ,
Rebounds,2006,100754 ,
Rebounds,2005,102970 ,
Rebounds,2004,100361 ,
Rebounds,2003,100604 ,
Rebounds,2002,100829 ,
Rebounds,2001,100988 ,
Rebounds,2000,102062 ,
Rebounds,1999,60395 ,
Rebounds,1998,98798 ,
Rebounds,1997,97703 ,
Rebounds,1996,98099 ,
Rebounds,1995,92006 ,
Rebounds,1994,95192 ,
Rebounds,1993,95504 ,
Rebounds,1992,96680 ,
Rebounds,1991,95776 ,
Rebounds,1990,95518 ,
Rebounds,1989,90031 ,
Rebounds,1988,81828 ,
Rebounds,1987,83020 ,
Rebounds,1986,82161 ,
Rebounds,1985,82008 ,
Rebounds,1984,81150 ,
Rebounds,1983,83853 ,
Rebounds,1982,81987 ,
Rebounds,1981,82010 ,
Rebounds,1980,81065 ,
Field Goals Attempted,2022,216722,
Field Goals Attempted,2021,190983,
Field Goals Attempted,2020,188116,
Field Goals Attempted,2019,219458,
Field Goals Attempted,2018,211709,
Field Goals Attempted,2017,210115,
Field Goals Attempted,2016,208049,
Field Goals Attempted,2015,205570,
Field Goals Attempted,2014,204172,
Field Goals Attempted,2013,201609,
Field Goals Attempted,2012,161225,
Field Goals Attempted,2011,199790,
Field Goals Attempted,2010,200989,
Field Goals Attempted,2009,199054,
Field Goals Attempted,2008,200501,
Field Goals Attempted,2007,196073,
Field Goals Attempted,2006,194315,
Field Goals Attempted,2005,197626,
Field Goals Attempted,2004,189802,
Field Goals Attempted,2003,192109,
Field Goals Attempted,2002,193263,
Field Goals Attempted,2001,191664,
Field Goals Attempted,2000,195220,
Field Goals Attempted,1999,113379,
Field Goals Attempted,1998,189537,
Field Goals Attempted,1997,188594,
Field Goals Attempted,1996,190675,
Field Goals Attempted,1995,180423,
Field Goals Attempted,1994,186951,
Field Goals Attempted,1993,190295,
Field Goals Attempted,1992,193391,
Field Goals Attempted,1991,193059,
Field Goals Attempted,1990,192951,
Field Goals Attempted,1989,182385,
Field Goals Attempted,1988,165441,
Field Goals Attempted,1987,167455,
Field Goals Attempted,1986,167166,
Field Goals Attempted,1985,168048,
Field Goals Attempted,1984,166638,
Field Goals Attempted,1983,169098,
Field Goals Attempted,1982,166418,
Field Goals Attempted,1981,166769,
Field Goals Attempted,1980,163521,
Field Goals Made,2022,99930,
Field Goals Made,2021,89020,
Field Goals Made,2020,86550,
Field Goals Made,2019,101062,
Field Goals Made,2018,97435,
Field Goals Made,2017,96061,
Field Goals Made,2016,94065,
Field Goals Made,2015,92287,
Field Goals Made,2014,92779,
Field Goals Made,2013,91282,
Field Goals Made,2012,72218,
Field Goals Made,2011,91624,
Field Goals Made,2010,92730,
Field Goals Made,2009,91310,
Field Goals Made,2008,91669,
Field Goals Made,2007,89860,
Field Goals Made,2006,88166,
Field Goals Made,2005,88435,
Field Goals Made,2004,83254,
Field Goals Made,2003,84937,
Field Goals Made,2002,86011,
Field Goals Made,2001,84865,
Field Goals Made,2000,87591,
Field Goals Made,1999,49549,
Field Goals Made,1998,85383,
Field Goals Made,1997,85777,
Field Goals Made,1996,88096,
Field Goals Made,1995,84105,
Field Goals Made,1994,87064,
Field Goals Made,1993,90056,
Field Goals Made,1992,91371,
Field Goals Made,1991,91551,
Field Goals Made,1990,91914,
Field Goals Made,1989,87060,
Field Goals Made,1988,79473,
Field Goals Made,1987,80422,
Field Goals Made,1986,81465,
Field Goals Made,1985,82532,
Field Goals Made,1984,82007,
Field Goals Made,1983,82087,
Field Goals Made,1982,81732,
Field Goals Made,1981,81025,
Field Goals Made,1980,78735

"""
        |> fromCSV
````

```elm {l=hidden}
threePointData : Table
threePointData =
    """
StatType,Season,Stat

3 Point Shots Attempted,2022,86535
3 Point Shots Attempted,2021,74822
3 Point Shots Attempted,2020,72252
3 Point Shots Attempted,2019,78742
3 Point Shots Attempted,2018,71340
3 Point Shots Attempted,2017,66422
3 Point Shots Attempted,2016,59241
3 Point Shots Attempted,2015,55137
3 Point Shots Attempted,2014,52974
3 Point Shots Attempted,2013,49067
3 Point Shots Attempted,2012,36395
3 Point Shots Attempted,2011,44313
3 Point Shots Attempted,2010,44622
3 Point Shots Attempted,2009,44583
3 Point Shots Attempted,2008,44544
3 Point Shots Attempted,2007,41672
3 Point Shots Attempted,2006,39313
3 Point Shots Attempted,2005,38748
3 Point Shots Attempted,2004,35492
3 Point Shots Attempted,2003,34912
3 Point Shots Attempted,2002,35074
3 Point Shots Attempted,2001,32597
3 Point Shots Attempted,2000,32614
3 Point Shots Attempted,1999,19080
3 Point Shots Attempted,1998,30231
3 Point Shots Attempted,1997,39943
3 Point Shots Attempted,1996,38161
3 Point Shots Attempted,1995,33889
3 Point Shots Attempted,1994,21907
3 Point Shots Attempted,1993,19824
3 Point Shots Attempted,1992,16898
3 Point Shots Attempted,1991,15812
3 Point Shots Attempted,1990,14608
3 Point Shots Attempted,1989,13431
3 Point Shots Attempted,1988,9421
3 Point Shots Attempted,1987,8913
3 Point Shots Attempted,1986,6293
3 Point Shots Attempted,1985,5917
3 Point Shots Attempted,1984,4484
3 Point Shots Attempted,1983,4248
3 Point Shots Attempted,1982,4308
3 Point Shots Attempted,1981,3815
3 Point Shots Attempted,1980,5003
3 Point Shots Made,2022,30598
3 Point Shots Made,2021,27427
3 Point Shots Made,2020,25862
3 Point Shots Made,2019,27955
3 Point Shots Made,2018,25807
3 Point Shots Made,2017,23748
3 Point Shots Made,2016,20953
3 Point Shots Made,2015,19300
3 Point Shots Made,2014,19054
3 Point Shots Made,2013,17603
3 Point Shots Made,2012,12693
3 Point Shots Made,2011,15886
3 Point Shots Made,2010,15822
3 Point Shots Made,2009,16352
3 Point Shots Made,2008,16124
3 Point Shots Made,2007,14926
3 Point Shots Made,2006,14086
3 Point Shots Made,2005,13777
3 Point Shots Made,2004,12321
3 Point Shots Made,2003,12200
3 Point Shots Made,2002,12402
3 Point Shots Made,2001,11524
3 Point Shots Made,2000,11513
3 Point Shots Made,1999,6463
3 Point Shots Made,1998,10450
3 Point Shots Made,1997,14383
3 Point Shots Made,1996,14000
3 Point Shots Made,1995,12153
3 Point Shots Made,1994,7301
3 Point Shots Made,1993,6668
3 Point Shots Made,1992,5587
3 Point Shots Made,1991,5055
3 Point Shots Made,1990,4829
3 Point Shots Made,1989,4332
3 Point Shots Made,1988,2979
3 Point Shots Made,1987,2687
3 Point Shots Made,1986,1774
3 Point Shots Made,1985,1671
3 Point Shots Made,1984,1120
3 Point Shots Made,1983,1011
3 Point Shots Made,1982,1129
3 Point Shots Made,1981,936
3 Point Shots Made,1980,1403
"""
        |> fromCSV
```

```elm {l=hidden}
salaryTable : Table
salaryTable =
    """
StatType,Season,AvarageYearlySalaryCap
Salary,1985,"9793069"
Salary,1986,"11298280"
Salary,1987,"12734751"
Salary,1988,"15248702"
Salary,1989,"17069461"
Salary,1990,"21950887"
Salary,1991,"25499592"
Salary,1992,"26061180"
Salary,1993,"28354756"
Salary,1994,"29954782"
Salary,1995,"30651995"
Salary,1996,"42906004"
Salary,1997,"44402880"
Salary,1998,"48285994"
Salary,1999,"52688773"
Salary,2000,"57763864"
Salary,2001,"58657395"
Salary,2002,"69132284"
Salary,2003,"64048956"
Salary,2004,"67907202"
Salary,2005,"65719039"
Salary,2006,"71839551"
Salary,2007,"74973988"
Salary,2008,"75594893"
Salary,2009,"80023973"
Salary,2010,"77414680"
Salary,2011,"75491545"
Salary,2012,"73970553"
Salary,2013,"72903264"
Salary,2014,"72519787"
Salary,2015,"77841847"
Salary,2016,"85323721"
Salary,2017,"112368348"
Salary,2018,"115455294"
Salary,2019,"116573245"
Salary,2020,"123384194"
Salary,2021,"117838496"
Salary,2022,"112414000"
"""
        |> fromCSV
```

```elm {l=hidden}
threePointStats : Table
threePointStats =
    """
StatType,Season,Stat
3 Point Shots Attempted,2022,86535
3 Point Shots Attempted,2021,74822
3 Point Shots Attempted,2020,72252
3 Point Shots Attempted,2019,78742
3 Point Shots Attempted,2018,71340
3 Point Shots Attempted,2017,66422
3 Point Shots Attempted,2016,59241
3 Point Shots Attempted,2015,55137
3 Point Shots Attempted,2014,52974
3 Point Shots Attempted,2013,49067
3 Point Shots Attempted,2012,36395
3 Point Shots Attempted,2011,44313
3 Point Shots Attempted,2010,44622
3 Point Shots Attempted,2009,44583
3 Point Shots Attempted,2008,44544
3 Point Shots Attempted,2007,41672
3 Point Shots Attempted,2006,39313
3 Point Shots Attempted,2005,38748
3 Point Shots Attempted,2004,35492
3 Point Shots Attempted,2003,34912
3 Point Shots Attempted,2002,35074
3 Point Shots Attempted,2001,32597
3 Point Shots Attempted,2000,32614
3 Point Shots Attempted,1999,19080
3 Point Shots Attempted,1998,30231
3 Point Shots Attempted,1997,39943
3 Point Shots Attempted,1996,38161
3 Point Shots Attempted,1995,33889
3 Point Shots Attempted,1994,21907
3 Point Shots Attempted,1993,19824
3 Point Shots Attempted,1992,16898
3 Point Shots Attempted,1991,15812
3 Point Shots Attempted,1990,14608
3 Point Shots Attempted,1989,13431
3 Point Shots Attempted,1988,9421
3 Point Shots Attempted,1987,8913
3 Point Shots Attempted,1986,6293
3 Point Shots Attempted,1985,5917
3 Point Shots Attempted,1984,4484
3 Point Shots Attempted,1983,4248
3 Point Shots Attempted,1982,4308
3 Point Shots Attempted,1981,3815
3 Point Shots Attempted,1980,5003
3 Point Shots Made,2022,30598
3 Point Shots Made,2021,27427
3 Point Shots Made,2020,25862
3 Point Shots Made,2019,27955
3 Point Shots Made,2018,25807
3 Point Shots Made,2017,23748
3 Point Shots Made,2016,20953
3 Point Shots Made,2015,19300
3 Point Shots Made,2014,19054
3 Point Shots Made,2013,17603
3 Point Shots Made,2012,12693
3 Point Shots Made,2011,15886
3 Point Shots Made,2010,15822
3 Point Shots Made,2009,16352
3 Point Shots Made,2008,16124
3 Point Shots Made,2007,14926
3 Point Shots Made,2006,14086
3 Point Shots Made,2005,13777
3 Point Shots Made,2004,12321
3 Point Shots Made,2003,12200
3 Point Shots Made,2002,12402
3 Point Shots Made,2001,11524
3 Point Shots Made,2000,11513
3 Point Shots Made,1999,6463
3 Point Shots Made,1998,10450
3 Point Shots Made,1997,14383
3 Point Shots Made,1996,14000
3 Point Shots Made,1995,12153
3 Point Shots Made,1994,7301
3 Point Shots Made,1993,6668
3 Point Shots Made,1992,5587
3 Point Shots Made,1991,5055
3 Point Shots Made,1990,4829
3 Point Shots Made,1989,4332
3 Point Shots Made,1988,2979
3 Point Shots Made,1987,2687
3 Point Shots Made,1986,1774
3 Point Shots Made,1985,1671
3 Point Shots Made,1984,1120
3 Point Shots Made,1983,1011
3 Point Shots Made,1982,1129
3 Point Shots Made,1981,936
3 Point Shots Made,1980,1403
3 Point Percentage,2022,35.4
3 Point Percentage,2021,36.7
3 Point Percentage,2020,35.8
3 Point Percentage,2019,35.5
3 Point Percentage,2018,36.2
3 Point Percentage,2017,35.8
3 Point Percentage,2016,35.4
3 Point Percentage,2015,35
3 Point Percentage,2014,36
3 Point Percentage,2013,35.9
3 Point Percentage,2012,34.9
3 Point Percentage,2011,35.8
3 Point Percentage,2010,35.5
3 Point Percentage,2009,36.7
3 Point Percentage,2008,36.2
3 Point Percentage,2007,35.8
3 Point Percentage,2006,35.8
3 Point Percentage,2005,35.6
3 Point Percentage,2004,34.7
3 Point Percentage,2003,34.9
3 Point Percentage,2002,35.4
3 Point Percentage,2001,35.4
3 Point Percentage,2000,35.3
3 Point Percentage,1999,33.9
3 Point Percentage,1998,34.6
3 Point Percentage,1997,36
3 Point Percentage,1996,36.7
3 Point Percentage,1995,35.9
3 Point Percentage,1994,33.3
3 Point Percentage,1993,33.6
3 Point Percentage,1992,33.1
3 Point Percentage,1991,32
3 Point Percentage,1990,33.1
3 Point Percentage,1989,32.3
3 Point Percentage,1988,31.6
3 Point Percentage,1987,30.1
3 Point Percentage,1986,28.2
3 Point Percentage,1985,28.2
3 Point Percentage,1984,25
3 Point Percentage,1983,23.8
3 Point Percentage,1982,26.2
3 Point Percentage,1981,24.5
3 Point Percentage,1980,28
Field Goal Percentage,2022,46.1
Field Goal Percentage,2021,46.6
Field Goal Percentage,2020,46
Field Goal Percentage,2019,46.1
Field Goal Percentage,2018,46
Field Goal Percentage,2017,45.7
Field Goal Percentage,2016,45.2
Field Goal Percentage,2015,44.9
Field Goal Percentage,2014,45.4
Field Goal Percentage,2013,45.3
Field Goal Percentage,2012,44.8
Field Goal Percentage,2011,45.9
Field Goal Percentage,2010,46.1
Field Goal Percentage,2009,45.9
Field Goal Percentage,2008,45.7
Field Goal Percentage,2007,45.8
Field Goal Percentage,2006,45.4
Field Goal Percentage,2005,44.7
Field Goal Percentage,2004,43.9
Field Goal Percentage,2003,44.2
Field Goal Percentage,2002,44.5
Field Goal Percentage,2001,44.3
Field Goal Percentage,2000,44.9
Field Goal Percentage,1999,43.7
Field Goal Percentage,1998,45
Field Goal Percentage,1997,45.5
Field Goal Percentage,1996,46.2
Field Goal Percentage,1995,46.6
Field Goal Percentage,1994,46.6
Field Goal Percentage,1993,47.3
Field Goal Percentage,1992,47.2
Field Goal Percentage,1991,47.4
Field Goal Percentage,1990,47.6
Field Goal Percentage,1989,47.7
Field Goal Percentage,1988,48
Field Goal Percentage,1987,48
Field Goal Percentage,1986,48.7
Field Goal Percentage,1985,49.1
Field Goal Percentage,1984,49.2
Field Goal Percentage,1983,48.5
Field Goal Percentage,1982,49.1
Field Goal Percentage,1981,48.6
Field Goal Percentage,1980,48.1
"""
        |> fromCSV
```

```elm {l=hidden}
threePointPositionData : Table
threePointPositionData =
    """

Season,Position,Stat
2022,Point Guard,20414
2022,Shooting Guard,29946
2022,Small Forward,18649
2022,Power Forward,12128
2022,Center,5288
2021,Point Guard,17608
2021,Shooting Guard,25536
2021,Small Forward,12275
2021,Power Forward,14577
2021,Center,4826
2020,Point Guard,16470
2020,Shooting Guard,21558
2020,Small Forward,14653
2020,Power Forward,14602
2020,Center,4969
2019,Point Guard,19529
2019,Shooting Guard,22298
2019,Small Forward,18299
2019,Power Forward,13945
2019,Center,4671
2018,Point Guard,15954
2018,Shooting Guard,21203
2018,Small Forward,15746
2018,Power Forward,13563
2018,Center,4873
2017,Point Guard,16375
2017,Shooting Guard,19751
2017,Small Forward,15109
2017,Power Forward,11143
2017,Center,4044
2016,Point Guard,14598
2016,Shooting Guard,17444
2016,Small Forward,16361
2016,Power Forward,8837
2016,Center,2001
2015,Point Guard,15102
2015,Shooting Guard,16440
2015,Small Forward,14743
2015,Power Forward,7595
2015,Center,1257
2014,Point Guard,14303
2014,Shooting Guard,16017
2014,Small Forward,13968
2014,Power Forward,7348
2014,Center,1338
2013,Point Guard,13272
2013,Shooting Guard,15745
2013,Small Forward,13637
2013,Power Forward,5922
2013,Center,491
2012,Point Guard,9732
2012,Shooting Guard,12560
2012,Small Forward,9888
2012,Power Forward,3772
2012,Center,443
2011,Point Guard,11573
2011,Shooting Guard,15688
2011,Small Forward,11929
2011,Power Forward,4539
2011,Center,584
2010,Point Guard,12302
2010,Shooting Guard,13665
2010,Small Forward,11474
2010,Power Forward,5722
2010,Center,1459
2009,Point Guard,11081
2009,Shooting Guard,14815
2009,Small Forward,12041
2009,Power Forward,5566
2009,Center,1080
2008,Point Guard,11113
2008,Shooting Guard,15421
2008,Small Forward,11345
2008,Power Forward,5617
2008,Center,1048
2007,Point Guard,11266
2007,Shooting Guard,12964
2007,Small Forward,11092
2007,Power Forward,5234
2007,Center,1116
2006,Point Guard,11861
2006,Shooting Guard,12369
2006,Small Forward,8968
2006,Power Forward,4868
2006,Center,1247
2005,Point Guard,11744
2005,Shooting Guard,11634
2005,Small Forward,10086
2005,Power Forward,4496
2005,Center,788
2004,Point Guard,10353
2004,Shooting Guard,11671
2004,Small Forward,9081
2004,Power Forward,3821
2004,Center,566
2003,Point Guard,10241
2003,Shooting Guard,10976
2003,Small Forward,9179
2003,Power Forward,4017
2003,Center,499
2002,Point Guard,10851
2002,Shooting Guard,11658
2002,Small Forward,8201
2002,Power Forward,3479
2002,Center,885
2001,Point Guard,10079
2001,Shooting Guard,10249
2001,Small Forward,8532
2001,Power Forward,3382
2001,Center,355
2000,Point Guard,10061
2000,Shooting Guard,10452
2000,Small Forward,8237
2000,Power Forward,3257
2000,Center,607
"""
        |> fromCSV
```
