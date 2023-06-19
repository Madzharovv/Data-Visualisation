---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/teaching.yml

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

https://github.com/Madzharovv/Year-1-Web-Development/blob/main/Resources/TheGraph1Sorted.csv
How much have Points, Assists, Rebounds metrics actually changed over time?

```elm {v }
multiLines : Spec
multiLines =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" totalsTableSorted |> nums)
                << dataColumn "Stat" (numColumn "Stat" totalsTableSorted |> nums)
                << dataColumn "StatType" (strColumn "StatType" totalsTableSorted |> strs)

        enc =
            encoding
                << position X [ pName "Season", pTitle "" ]
                << position Y [ pName "Stat", pQuant ]
                << color [ mName "StatType", mTitle "StatType", mScale [ scScheme "dark2" [] ] ]
    in
    toVegaLite [ width 400, data [], enc [], line [] ]
```

```elm {v interactive}
dynamicTooltips : Spec
dynamicTooltips =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" totalsTableSorted |> nums)
                << dataColumn "Stat" (numColumn "Stat" totalsTableSorted |> nums)
                << dataColumn "StatType" (strColumn "StatType" totalsTableSorted |> strs)

        enc =
            encoding
                << position X [ pName "Season", pTitle "Season" ]

        transSelFilter =
            transform
                << filter (fiSelectionEmpty "hover")

        enc1 =
            encoding
                << position Y [ pName "Stat", pQuant ]
                << color [ mName "StatType", mTitle "" ]

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
                    [ [ tName "TRB", tQuant ]
                    , [ tName "PTS", tQuant ]
                    , [ tName "AST", tQuant ]
                    , [ tName "STL", tQuant ]
                    , [ tName "TOV", tQuant ]
                    , [ tName "PF", tQuant ]
                    , [ tName "BLK", tQuant ]
                    , [ tName "FG", tQuant ]
                    , [ tName "FGA", tQuant ]
                    , [ tName "Season", tQuant ]
                    ]

        spec2 =
            asSpec [ ps [], transPivot [], enc2 [], rule [] ]
    in
    toVegaLite [ width 600, height 300, data [], enc [], layer [ spec1, spec2 ] ]
```

```elm {v interactive}
dynamicTooltipsAVG : Spec
dynamicTooltipsAVG =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" totalsTableSorted2 |> nums)
                << dataColumn "Stat" (numColumn "Stat" totalsTableSorted2 |> nums)
                << dataColumn "StatType" (strColumn "StatType" totalsTableSorted2 |> strs)

        enc =
            encoding
                << position X [ pName "Season", pTitle "Season", pAxis [ axLabelAngle 0 ] ]

        transSelFilter =
            transform
                << filter (fiSelectionEmpty "hover")

        enc1 =
            encoding
                << position Y [ pName "Stat", pQuant ]
                << color [ mName "StatType", mTitle "", mScale [ scScheme "dark2" [] ] ]

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
                    [ [ tName "TRB", tQuant, tTitle "Rebounds" ]
                    , [ tName "PTS", tQuant, tTitle "Points" ]
                    , [ tName "AST", tQuant, tTitle "Assists" ]
                    , [ tName "STL", tQuant, tTitle "Steals" ]
                    , [ tName "TOV", tQuant, tTitle "Turnovers" ]
                    , [ tName "PF", tQuant, tTitle "Personal Fouls" ]
                    , [ tName "BLK", tQuant, tTitle "Blocks" ]
                    , [ tName "FG", tQuant, tTitle "Field Goals" ]
                    , [ tName "FGA", tQuant, tTitle "Field Goals" ]
                    , [ tName "Season", tQuant, tTitle "Rebounds" ]
                    ]

        spec2 =
            asSpec [ ps [], transPivot [], enc2 [], rule [] ]

        cfg =
            configure
                -- << configuration (coBackground "grey")
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite [ width 1200, height 1000, data [], enc [], cfg [], layer [ spec1, spec2 ] ]
```


```elm {v interactive}
interactiveHighlight : Spec
interactiveHighlight =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" totalsTableSorted2 |> nums)
                << dataColumn "Stat" (numColumn "Stat" totalsTableSorted2 |> nums)
                << dataColumn "StatType" (strColumn "StatType" totalsTableSorted2 |> strs)
        ps =
            params
                << param "myHover"
                    [ paSelect sePoint [ seOn "mouseover", seFields [ "StatType" ] ]
                    , paValues (dataObjects [ [ ( "StatType", str "TRB" ) ] ])
                    ]

        trans =
            transform
                << filter (fiExpr "datum.symbol !== 'IBM'")

        enc =
            encoding
                << color
                    [ mCondition (prParamEmpty "myHover")
                        [ mName "StatType", mLegend [] ]
                        [ mStr "grey" ]
                    ]
                << opacity
                    [ mCondition (prParamEmpty "myHover")
                        [ mNum 1 ]
                        [ mNum 0.2 ]
                    ]

        enc1 =
            encoding
                << position X [ pName "Season", pTitle "" ]
                << position Y [ pName "Stat", pQuant, pTitle "Stat" ]

        spec1 =
            --Transparent layer to make it easier to trigger selection
            asSpec
                [ enc1 []
                , layer
                    [ asSpec [ ps [], line [ maStrokeWidth 8, maStroke "transparent" ] ]
                    , asSpec [ line [] ]
                    ]
                ]

        enc2 =
            encoding
                << position X [ pName "Season" ]
                << position Y [ pName "Stat", pQuant]

        enc2_1 =
            encoding
                << text [ tName "StatType" ]

        spec2 =
            asSpec
                [ enc2 []
                , layer
                    [ asSpec [ circle [] ]
                    , asSpec [ enc2_1 [], textMark[] ]
                    ]
                ]
    in
    toVegaLite [ width 1400, height 1000,cfg [], data[], trans [], enc [], layer [ spec1, spec2 ] ]
```
axis prop

















```elm {v}
lineChartWithMarkers : Spec
lineChartWithMarkers =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" salaryTable |> nums)
                << dataColumn "AvarageYearlySalaryCap" (numColumn "AvarageYearlySalaryCap" salaryTable |> nums)

        enc =
            encoding
                << position X [ pName "Season" ]
                << position Y [ pName "AvarageYearlySalaryCap", pQuant, pTitle "Avarage Yearly Salary Cap in 2022 Dollars considering Inflation" ]
                << shape [ mName "Origin", mScale [] ]
    in
    toVegaLite [ width 1000, height 700, data [], enc [], line [ maFilled True, maSize 200, maOpacity 0.5 ] ]
```

categorical domain

```elm {v interactive}
dynamicTooltipsAVG2 : Spec
dynamicTooltipsAVG2 =
    let
        data =
            dataFromColumns []
                << dataColumn "Season" (numColumn "Season" threePointData |> nums)
                << dataColumn "Stat" (numColumn "Stat" threePointData |> nums)
                << dataColumn "StatType" (strColumn "StatType" threePointData |> strs)

        enc =
            encoding
                << position X [ pName "Season", pTitle "Season" ]

        transSelFilter =
            transform
                << filter (fiSelectionEmpty "hover")

        enc1 =
            encoding
                << position Y [ pName "Stat", pQuant ]
                << color [ mName "StatType", mTitle "", mScale [ scScheme "dark2" [] ] ]

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
                    [ [ tName "3P", tQuant ]
                    , [ tName "3PA", tQuant ]
                    ]

        spec2 =
            asSpec [ ps [], transPivot [], enc2 [], rule [] ]

        cfg =
            configure
                -- << configuration (coBackground "grey")
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite [ width 1200, height 1000, data [], enc [], cfg [], layer [ spec1, spec2 ] ]
```
session 7 layoutr 
supperposittion
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
threePointData : Table
threePointData =
    """
StatType,Season,Stat

3PA,2022,86535
3PA,2021,74822
3PA,2020,72252
3PA,2019,78742
3PA,2018,71340
3PA,2017,66422
3PA,2016,59241
3PA,2015,55137
3PA,2014,52974
3PA,2013,49067
3PA,2012,36395
3PA,2011,44313
3PA,2010,44622
3PA,2009,44583
3PA,2008,44544
3PA,2007,41672
3PA,2006,39313
3PA,2005,38748
3PA,2004,35492
3PA,2003,34912
3PA,2002,35074
3PA,2001,32597
3PA,2000,32614
3PA,1999,19080
3PA,1998,30231
3PA,1997,39943
3PA,1996,38161
3PA,1995,33889
3PA,1994,21907
3PA,1993,19824
3PA,1992,16898
3PA,1991,15812
3PA,1990,14608
3PA,1989,13431
3PA,1988,9421
3PA,1987,8913
3PA,1986,6293
3PA,1985,5917
3PA,1984,4484
3PA,1983,4248
3PA,1982,4308
3PA,1981,3815
3PA,1980,5003
3P,2022,30598
3P,2021,27427
3P,2020,25862
3P,2019,27955
3P,2018,25807
3P,2017,23748
3P,2016,20953
3P,2015,19300
3P,2014,19054
3P,2013,17603
3P,2012,12693
3P,2011,15886
3P,2010,15822
3P,2009,16352
3P,2008,16124
3P,2007,14926
3P,2006,14086
3P,2005,13777
3P,2004,12321
3P,2003,12200
3P,2002,12402
3P,2001,11524
3P,2000,11513
3P,1999,6463
3P,1998,10450
3P,1997,14383
3P,1996,14000
3P,1995,12153
3P,1994,7301
3P,1993,6668
3P,1992,5587
3P,1991,5055
3P,1990,4829
3P,1989,4332
3P,1988,2979
3P,1987,2687
3P,1986,1774
3P,1985,1671
3P,1984,1120
3P,1983,1011
3P,1982,1129
3P,1981,936
3P,1980,1403
"""
        |> fromCSV
```

<!-- ```elm {m}
displaytotalsTable : List String
displaytotalsTable =
    totalsTableSorted
        |> tableSummary 440
``` -->

```elm {l=hidden}
totalsTableSorted : Table
totalsTableSorted =
    """

StatType,Season,Stat,

AST,2022,60636,                         
AST,2021,53577 ,
AST,2020,51649 ,
AST,2019,60483 ,
AST,2018,57163 ,
AST,2017,55660 ,
AST,2016,54832 ,
AST,2015,54202 ,
AST,2014,54131 ,
AST,2013,54415 ,
AST,2012,41533 ,
AST,2011,52887 ,
AST,2010,52258 ,
AST,2009,51605 ,
AST,2008,53506 ,
AST,2007,52365 ,
AST,2006,50711 ,
AST,2005,52342 ,
AST,2004,50658 ,
AST,2003,51133 ,
AST,2002,52131 ,
AST,2001,51786 ,
AST,2000,53130 ,
AST,1999,30050 ,
AST,1998,52386 ,
AST,1997,52426 ,
AST,1996,53950 ,
AST,1995,51729 ,
AST,1994,54003 ,
AST,1993,54705 ,
AST,1992,54192 ,
AST,1991,54782 ,
AST,1990,55023 ,
AST,1989,52431 ,
AST,1988,48599 ,
AST,1987,48998 ,
AST,1986,49065 ,
AST,1985,49515 ,
AST,1984,49397 ,
AST,1983,48862 ,
AST,1982,47438 ,
AST,1981,48034 ,
AST,1980,46567 ,
BLK,2022,11594 ,
BLK,2021,10525 ,
BLK,2020,10378 ,
BLK,2019,12185 ,
BLK,2018,11846 ,
BLK,2017,11669 ,
BLK,2016,12193 ,
BLK,2015,11797 ,
BLK,2014,11594 ,
BLK,2013,12622 ,
BLK,2012,10086 ,
BLK,2011,11966 ,
BLK,2010,11947 ,
BLK,2009,11817 ,
BLK,2008,11651 ,
BLK,2007,11329 ,
BLK,2006,11560 ,
BLK,2005,11963 ,
BLK,2004,12022 ,
BLK,2003,11928 ,
BLK,2002,12422 ,
BLK,2001,12505 ,
BLK,2000,12293 ,
BLK,1999,7196 ,
BLK,1998,12062 ,
BLK,1997,11674 ,
BLK,1996,12038 ,
BLK,1995,11430 ,
BLK,1994,11579 ,
BLK,1993,11558 ,
BLK,1992,12216 ,
BLK,1991,11624 ,
BLK,1990,11207 ,
BLK,1989,10949 ,
BLK,1988,10160 ,
BLK,1987,10411 ,
BLK,1986,9910 ,
BLK,1985,10019 ,
BLK,1984,10004 ,
BLK,1983,10566 ,
BLK,1982,10131 ,
BLK,1981,10036 ,
BLK,1980,9561 ,
PF,2022,48306 ,
PF,2021,41669 ,
PF,2020,44004 ,
PF,2019,51425 ,
PF,2018,48837 ,
PF,2017,48950 ,
PF,2016,49854 ,
PF,2015,49728 ,
PF,2014,50923 ,
PF,2013,48775 ,
PF,2012,38744 ,
PF,2011,50953 ,
PF,2010,51309 ,
PF,2009,51765 ,
PF,2008,51709 ,
PF,2007,54666 ,
PF,2006,55986 ,
PF,2005,55671 ,
PF,2004,51006 ,
PF,2003,51730 ,
PF,2002,50497 ,
PF,2001,53143 ,
PF,2000,55408 ,
PF,1999,32206 ,
PF,1998,53278 ,
PF,1997,52602 ,
PF,1996,54806 ,
PF,1995,51961 ,
PF,1994,49075 ,
PF,1993,51269 ,
PF,1992,49227 ,
PF,1991,51304 ,
PF,1990,51478 ,
PF,1989,48504 ,
PF,1988,45461 ,
PF,1987,46277 ,
PF,1986,47543 ,
PF,1985,47036 ,
PF,1984,48573 ,
PF,1983,48373 ,
PF,1982,49352 ,
PF,1981,47265 ,
PF,1980,43952 ,
PTS,2022,272115 ,
PTS,2021,242117 ,
PTS,2020,236788 ,
PTS,2019,273573 ,
PTS,2018,261580 ,
PTS,2017,259753 ,
PTS,2016,252572 ,
PTS,2015,246035 ,
PTS,2014,248482 ,
PTS,2013,241223 ,
PTS,2012,190594 ,
PTS,2011,244894 ,
PTS,2010,247100 ,
PTS,2009,245879 ,
PTS,2008,245811 ,
PTS,2007,242899 ,
PTS,2006,238641 ,
PTS,2005,239109 ,
PTS,2004,222097 ,
PTS,2003,226102 ,
PTS,2002,227043 ,
PTS,2001,225459 ,
PTS,2000,231789 ,
PTS,1999,132792 ,
PTS,1998,227269 ,
PTS,1997,230433 ,
PTS,1996,236619 ,
PTS,1995,224518 ,
PTS,1994,224737 ,
PTS,1993,233073 ,
PTS,1992,233155 ,
PTS,1991,235370 ,
PTS,1990,236882 ,
PTS,1989,223797 ,
PTS,1988,203993 ,
PTS,1987,207338 ,
PTS,1986,207875 ,
PTS,1984,209041 ,
PTS,1983,207668 ,
PTS,1982,204658 ,
PTS,1981,204776 ,
PTS,1980,203886 ,
PTS,1980,197239 ,
STL,2022,18772 ,
STL,2021,16356 ,
STL,2020,16200 ,
STL,2019,18779 ,
STL,2018,18983 ,
STL,2017,18950 ,
STL,2016,19303 ,
STL,2015,19031 ,
STL,2014,18895 ,
STL,2013,19178 ,
STL,2012,15200 ,
STL,2011,18023 ,
STL,2010,17759 ,
STL,2009,17883 ,
STL,2008,17897 ,
STL,2007,17807 ,
STL,2006,17632 ,
STL,2005,18489 ,
STL,2004,18864 ,
STL,2003,18884 ,
STL,2002,18538 ,
STL,2001,18600 ,
STL,2000,18868 ,
STL,1999,12113 ,
STL,1998,19943 ,
STL,1997,19486 ,
STL,1996,18952 ,
STL,1995,18330 ,
STL,1994,19652 ,
STL,1993,18932 ,
STL,1992,19131 ,
STL,1991,19014 ,
STL,1990,18881 ,
STL,1989,18619 ,
STL,1988,16055 ,
STL,1987,16264 ,
STL,1986,16576 ,
STL,1985,16122 ,
STL,1984,16041 ,
STL,1983,16778 ,
STL,1982,16107 ,
STL,1981,16964 ,
STL,1980,16993 ,
TOV,2022,33856 ,
TOV,2021,29887 ,
TOV,2020,30802 ,
TOV,2019,34644 ,
TOV,2018,35092 ,
TOV,2017,34320 ,
TOV,2016,35384 ,
TOV,2015,35312 ,
TOV,2014,36034 ,
TOV,2013,35767 ,
TOV,2012,28860 ,
TOV,2011,35065 ,
TOV,2010,34982 ,
TOV,2009,34519 ,
TOV,2008,34719 ,
TOV,2007,37232 ,
TOV,2006,35459 ,
TOV,2005,35677 ,
TOV,2004,35610 ,
TOV,2003,35471 ,
TOV,2002,34389 ,
TOV,2001,35783 ,
TOV,2000,36788 ,
TOV,1999,22221 ,
TOV,1998,36823 ,
TOV,1997,37252 ,
TOV,1996,37641 ,
TOV,1995,35303 ,
TOV,1994,35424 ,
TOV,1993,35230 ,
TOV,1992,34430 ,
TOV,1991,35515 ,
TOV,1990,35555 ,
TOV,1989,35312 ,
TOV,1988,31563 ,
TOV,1987,32040 ,
TOV,1986,33645 ,
TOV,1985,33694 ,
TOV,1984,33755 ,
TOV,1983,36038 ,
TOV,1982,33435 ,
TOV,1981,35354 ,
TOV,1980,34166 ,
TRB,2022,109347 ,
TRB,2021,95686 ,
TRB,2020,94957 ,
TRB,2019,111107 ,
TRB,2018,107052 ,
TRB,2017,107046 ,
TRB,2016,107645 ,
TRB,2015,106504 ,
TRB,2014,105161 ,
TRB,2013,103575 ,
TRB,2012,83513 ,
TRB,2011,101816 ,
TRB,2010,102640 ,
TRB,2009,101586 ,
TRB,2008,103271 ,
TRB,2007,100994 ,
TRB,2006,100754 ,
TRB,2005,102970 ,
TRB,2004,100361 ,
TRB,2003,100604 ,
TRB,2002,100829 ,
TRB,2001,100988 ,
TRB,2000,102062 ,
TRB,1999,60395 ,
TRB,1998,98798 ,
TRB,1997,97703 ,
TRB,1996,98099 ,
TRB,1995,92006 ,
TRB,1994,95192 ,
TRB,1993,95504 ,
TRB,1992,96680 ,
TRB,1991,95776 ,
TRB,1990,95518 ,
TRB,1989,90031 ,
TRB,1988,81828 ,
TRB,1987,83020 ,
TRB,1986,82161 ,
TRB,1985,82008 ,
TRB,1984,81150 ,
TRB,1983,83853 ,
TRB,1982,81987 ,
TRB,1981,82010 ,
TRB,1980,81065 ,
FGA,2022,216722,
FGA,2021,190983,
FGA,2020,188116,
FGA,2019,219458,
FGA,2018,211709,
FGA,2017,210115,
FGA,2016,208049,
FGA,2015,205570,
FGA,2014,204172,
FGA,2013,201609,
FGA,2012,161225,
FGA,2011,199790,
FGA,2010,200989,
FGA,2009,199054,
FGA,2008,200501,
FGA,2007,196073,
FGA,2006,194315,
FGA,2005,197626,
FGA,2004,189802,
FGA,2003,192109,
FGA,2002,193263,
FGA,2001,191664,
FGA,2000,195220,
FGA,1999,113379,
FGA,1998,189537,
FGA,1997,188594,
FGA,1996,190675,
FGA,1995,180423,
FGA,1994,186951,
FGA,1993,190295,
FGA,1992,193391,
FGA,1991,193059,
FGA,1990,192951,
FGA,1989,182385,
FGA,1988,165441,
FGA,1987,167455,
FGA,1986,167166,
FGA,1985,168048,
FGA,1984,166638,
FGA,1983,169098,
FGA,1982,166418,
FGA,1981,166769,
FGA,1980,163521,
FG,2022,99930,
FG,2021,89020,
FG,2020,86550,
FG,2019,101062,
FG,2018,97435,
FG,2017,96061,
FG,2016,94065,
FG,2015,92287,
FG,2014,92779,
FG,2013,91282,
FG,2012,72218,
FG,2011,91624,
FG,2010,92730,
FG,2009,91310,
FG,2008,91669,
FG,2007,89860,
FG,2006,88166,
FG,2005,88435,
FG,2004,83254,
FG,2003,84937,
FG,2002,86011,
FG,2001,84865,
FG,2000,87591,
FG,1999,49549,
FG,1998,85383,
FG,1997,85777,
FG,1996,88096,
FG,1995,84105,
FG,1994,87064,
FG,1993,90056,
FG,1992,91371,
FG,1991,91551,
FG,1990,91914,
FG,1989,87060,
FG,1988,79473,
FG,1987,80422,
FG,1986,81465,
FG,1985,82532,
FG,1984,82007,
FG,1983,82087,
FG,1982,81732,
FG,1981,81025,
FG,1980,78735

"""
        |> fromCSV
```

<!--
```elm {m}
displaytotalsTable2 : List String
displaytotalsTable2 =
    totalsTableSorted2
        |> tableSummary 440
``` -->

```elm {l=hidden}
totalsTableSorted2 : Table
totalsTableSorted2 =
    """

StatType,Season,Stat,

AST,2022,60636,                         
AST,2021,60577 ,
AST,2020,60649 ,
AST,2019,60483 ,
AST,2018,57163 ,
AST,2017,55660 ,
AST,2016,54832 ,
AST,2015,54202 ,
AST,2014,54131 ,
AST,2013,54415 ,
AST,2012,53533 ,
AST,2011,52887 ,
AST,2010,52258 ,
AST,2009,51605 ,
AST,2008,53506 ,
AST,2007,52365 ,
AST,2006,50711 ,
AST,2005,52342 ,
AST,2004,50658 ,
AST,2003,51133 ,
AST,2002,52131 ,
AST,2001,51786 ,
AST,2000,53130 ,
AST,1999,52750 ,
AST,1998,52386 ,
AST,1997,52426 ,
AST,1996,53950 ,
AST,1995,51729 ,
AST,1994,54003 ,
AST,1993,54705 ,
AST,1992,54192 ,
AST,1991,54782 ,
AST,1990,55023 ,
AST,1989,52431 ,
AST,1988,48599 ,
AST,1987,48998 ,
AST,1986,49065 ,
AST,1985,49515 ,
AST,1984,49397 ,
AST,1983,48862 ,
AST,1982,47438 ,
AST,1981,48034 ,
AST,1980,46567 ,
BLK,2022,11594 ,
BLK,2021,11525 ,
BLK,2020,11378 ,
BLK,2019,12185 ,
BLK,2018,11846 ,
BLK,2017,11669 ,
BLK,2016,12193 ,
BLK,2015,11797 ,
BLK,2014,11594 ,
BLK,2013,12622 ,
BLK,2012,12086 ,
BLK,2011,11966 ,
BLK,2010,11947 ,
BLK,2009,11817 ,
BLK,2008,11651 ,
BLK,2007,11329 ,
BLK,2006,11560 ,
BLK,2005,11963 ,
BLK,2004,12022 ,
BLK,2003,11928 ,
BLK,2002,12422 ,
BLK,2001,12505 ,
BLK,2000,12293 ,
BLK,1999,12100 ,
BLK,1998,12062 ,
BLK,1997,11674 ,
BLK,1996,12038 ,
BLK,1995,11430 ,
BLK,1994,11579 ,
BLK,1993,11558 ,
BLK,1992,12216 ,
BLK,1991,11624 ,
BLK,1990,11207 ,
BLK,1989,10949 ,
BLK,1988,10160 ,
BLK,1987,10411 ,
BLK,1986,9910 ,
BLK,1985,10019 ,
BLK,1984,10004 ,
BLK,1983,10566 ,
BLK,1982,10131 ,
BLK,1981,10036 ,
BLK,1980,9561 ,
PF,2022,48306 ,
PF,2021,48669 ,
PF,2020,48004 ,
PF,2019,51425 ,
PF,2018,48837 ,
PF,2017,48950 ,
PF,2016,49854 ,
PF,2015,49728 ,
PF,2014,50923 ,
PF,2013,48775 ,
PF,2012,47744 ,
PF,2011,50953 ,
PF,2010,51309 ,
PF,2009,51765 ,
PF,2008,51709 ,
PF,2007,54666 ,
PF,2006,55986 ,
PF,2005,55671 ,
PF,2004,51006 ,
PF,2003,51730 ,
PF,2002,50497 ,
PF,2001,53143 ,
PF,2000,55408 ,
PF,1999,54206 ,
PF,1998,53278 ,
PF,1997,52602 ,
PF,1996,54806 ,
PF,1995,51961 ,
PF,1994,49075 ,
PF,1993,51269 ,
PF,1992,49227 ,
PF,1991,51304 ,
PF,1990,51478 ,
PF,1989,48504 ,
PF,1988,45461 ,
PF,1987,46277 ,
PF,1986,47543 ,
PF,1985,47036 ,
PF,1984,48573 ,
PF,1983,48373 ,
PF,1982,49352 ,
PF,1981,47265 ,
PF,1980,43952 ,
PTS,2022,272115 ,
PTS,2021,272117 ,
PTS,2020,276788 ,
PTS,2019,273573 ,
PTS,2018,261580 ,
PTS,2017,259753 ,
PTS,2016,252572 ,
PTS,2015,246035 ,
PTS,2014,248482 ,
PTS,2013,241223 ,
PTS,2012,246594 ,
PTS,2011,244894 ,
PTS,2010,247100 ,
PTS,2009,245879 ,
PTS,2008,245811 ,
PTS,2007,242899 ,
PTS,2006,238641 ,
PTS,2005,239109 ,
PTS,2004,222097 ,
PTS,2003,226102 ,
PTS,2002,227043 ,
PTS,2001,225459 ,
PTS,2000,231789 ,
PTS,1999,232792 ,
PTS,1998,227269 ,
PTS,1997,230433 ,
PTS,1996,236619 ,
PTS,1995,224518 ,
PTS,1994,224737 ,
PTS,1993,233073 ,
PTS,1992,233155 ,
PTS,1991,235370 ,
PTS,1990,236882 ,
PTS,1989,223797 ,
PTS,1988,203993 ,
PTS,1987,207338 ,
PTS,1986,207875 ,
PTS,1984,209041 ,
PTS,1983,207668 ,
PTS,1982,204658 ,
PTS,1981,204776 ,
PTS,1980,203886 ,
PTS,1980,197239 ,
STL,2022,18772 ,
STL,2021,18356 ,
STL,2020,18200 ,
STL,2019,18779 ,
STL,2018,18983 ,
STL,2017,18950 ,
STL,2016,19303 ,
STL,2015,19031 ,
STL,2014,18895 ,
STL,2013,19178 ,
STL,2012,18800 ,
STL,2011,18023 ,
STL,2010,17759 ,
STL,2009,17883 ,
STL,2008,17897 ,
STL,2007,17807 ,
STL,2006,17632 ,
STL,2005,18489 ,
STL,2004,18864 ,
STL,2003,18884 ,
STL,2002,18538 ,
STL,2001,18600 ,
STL,2000,18868 ,
STL,1999,17113 ,
STL,1998,19943 ,
STL,1997,19486 ,
STL,1996,18952 ,
STL,1995,18330 ,
STL,1994,19652 ,
STL,1993,18932 ,
STL,1992,19131 ,
STL,1991,19014 ,
STL,1990,18881 ,
STL,1989,18619 ,
STL,1988,16055 ,
STL,1987,16264 ,
STL,1986,16576 ,
STL,1985,16122 ,
STL,1984,16041 ,
STL,1983,16778 ,
STL,1982,16107 ,
STL,1981,16964 ,
STL,1980,16993 ,
TOV,2022,33856 ,
TOV,2021,32887 ,
TOV,2020,30802 ,
TOV,2019,34644 ,
TOV,2018,35092 ,
TOV,2017,34320 ,
TOV,2016,35384 ,
TOV,2015,35312 ,
TOV,2014,36034 ,
TOV,2013,35767 ,
TOV,2012,35560 ,
TOV,2011,35065 ,
TOV,2010,34982 ,
TOV,2009,34519 ,
TOV,2008,34719 ,
TOV,2007,37232 ,
TOV,2006,35459 ,
TOV,2005,35677 ,
TOV,2004,35610 ,
TOV,2003,35471 ,
TOV,2002,34389 ,
TOV,2001,35783 ,
TOV,2000,36788 ,
TOV,1999,36721 ,
TOV,1998,36823 ,
TOV,1997,37252 ,
TOV,1996,37641 ,
TOV,1995,35303 ,
TOV,1994,35424 ,
TOV,1993,35230 ,
TOV,1992,34430 ,
TOV,1991,35515 ,
TOV,1990,35555 ,
TOV,1989,35312 ,
TOV,1988,31563 ,
TOV,1987,32040 ,
TOV,1986,33645 ,
TOV,1985,33694 ,
TOV,1984,33755 ,
TOV,1983,36038 ,
TOV,1982,33435 ,
TOV,1981,35354 ,
TOV,1980,34166 ,
TRB,2022,109347 ,
TRB,2021,106860 ,
TRB,2020,109570 ,
TRB,2019,111107 ,
TRB,2018,107052 ,
TRB,2017,107046 ,
TRB,2016,107645 ,
TRB,2015,106504 ,
TRB,2014,105161 ,
TRB,2013,103575 ,
TRB,2012,102013 ,
TRB,2011,101816 ,
TRB,2010,102640 ,
TRB,2009,101586 ,
TRB,2008,103271 ,
TRB,2007,100994 ,
TRB,2006,100754 ,
TRB,2005,102970 ,
TRB,2004,100361 ,
TRB,2003,100604 ,
TRB,2002,100829 ,
TRB,2001,100988 ,
TRB,2000,102062 ,
TRB,1999,100395 ,
TRB,1998,98798 ,
TRB,1997,97703 ,
TRB,1996,98099 ,
TRB,1995,92006 ,
TRB,1994,95192 ,
TRB,1993,95504 ,
TRB,1992,96680 ,
TRB,1991,95776 ,
TRB,1990,95518 ,
TRB,1989,90031 ,
TRB,1988,81828 ,
TRB,1987,83020 ,
TRB,1986,82161 ,
TRB,1985,82008 ,
TRB,1984,81150 ,
TRB,1983,83853 ,
TRB,1982,81987 ,
TRB,1981,82010 ,
TRB,1980,81065 ,
FGA,2022,216722,
FGA,2021,210983,
FGA,2020,218116,
FGA,2019,219458,
FGA,2018,211709,
FGA,2017,210115,
FGA,2016,208049,
FGA,2015,205570,
FGA,2014,204172,
FGA,2013,201609,
FGA,2012,201225,
FGA,2011,199790,
FGA,2010,200989,
FGA,2009,199054,
FGA,2008,200501,
FGA,2007,196073,
FGA,2006,194315,
FGA,2005,197626,
FGA,2004,189802,
FGA,2003,192109,
FGA,2002,193263,
FGA,2001,191664,
FGA,2000,195220,
FGA,1999,183379,
FGA,1998,189537,
FGA,1997,188594,
FGA,1996,190675,
FGA,1995,180423,
FGA,1994,186951,
FGA,1993,190295,
FGA,1992,193391,
FGA,1991,193059,
FGA,1990,192951,
FGA,1989,182385,
FGA,1988,165441,
FGA,1987,167455,
FGA,1986,167166,
FGA,1985,168048,
FGA,1984,166638,
FGA,1983,169098,
FGA,1982,166418,
FGA,1981,166769,
FGA,1980,163521,
FG,2022,99930,
FG,2021,99020,
FG,2020,96550,
FG,2019,101062,
FG,2018,97435,
FG,2017,96061,
FG,2016,94065,
FG,2015,92287,
FG,2014,92779,
FG,2013,91282,
FG,2012,92218,
FG,2011,91624,
FG,2010,92730,
FG,2009,91310,
FG,2008,91669,
FG,2007,89860,
FG,2006,88166,
FG,2005,88435,
FG,2004,83254,
FG,2003,84937,
FG,2002,86011,
FG,2001,84865,
FG,2000,87591,
FG,1999,84549,
FG,1998,85383,
FG,1997,85777,
FG,1996,88096,
FG,1995,84105,
FG,1994,87064,
FG,1993,90056,
FG,1992,91371,
FG,1991,91551,
FG,1990,91914,
FG,1989,87060,
FG,1988,79473,
FG,1987,80422,
FG,1986,81465,
FG,1985,82532,
FG,1984,82007,
FG,1983,82087,
FG,1982,81732,
FG,1981,81025,
FG,1980,78735
"""
        |> fromCSV
```

```elm {l=hidden}
totalsTable2 : Table
totalsTable2 =
    """
Season,TRB,AST,STL,BLK,TOV,PF,PTS
2021-22,109347,60636,18772,11594,33856,48306,272115
2020-21,95686,53577,16356,10525,29887,41669,242117
2019-20,94957,51649,16200,10378,30802,44004,236788
2018-19,111107,60483,18779,12185,34644,51425,273573
2017-18,107052,57163,18983,11846,35092,48837,261580
2016-17,107046,55660,18950,11669,34320,48950,259753
2015-16,107645,54832,19303,12193,35384,49854,252572
2014-15,106504,54202,19031,11797,35312,49728,246035
2013-14,105161,54131,18895,11594,36034,50923,248482
2012-13,103575,54415,19178,12622,35767,48775,241223
2011-12,83513,41533,15200,10086,28860,38744,190594
2010-11,101816,52887,18023,11966,35065,50953,244894
2009-10,102640,52258,17759,11947,34982,51309,247100
2008-09,101586,51605,17883,11817,34519,51765,245879
2007-08,103271,53506,17897,11651,34719,51709,245811
2006-07,100994,52365,17807,11329,37232,54666,242899
2005-06,100754,50711,17632,11560,35459,55986,238641
2004-05,102970,52342,18489,11963,35677,55671,239109
2003-04,100361,50658,18864,12022,35610,51006,222097
2002-03,100604,51133,18884,11928,35471,51730,226102
2001-02,100829,52131,18538,12422,34389,50497,227043
2000-01,100988,51786,18600,12505,35783,53143,225459
1999-00,102062,53130,18868,12293,36788,55408,231789
1998-99,60395,30050,12113,7196,22221,32206,132792
1997-98,98798,52386,19943,12062,36823,53278,227269
1996-97,97703,52426,19486,11674,37252,52602,230433
1995-96,98099,53950,18952,12038,37641,54806,236619
1994-95,92006,51729,18330,11430,35303,51961,224518
1993-94,95192,54003,19652,11579,35424,49075,224737
1992-93,95504,54705,18932,11558,35230,51269,233073
1991-92,96680,54192,19131,12216,34430,49227,233155
1990-91,95776,54782,19014,11624,35515,51304,235370
1989-90,95518,55023,18881,11207,35555,51478,236882
1988-89,90031,52431,18619,10949,35312,48504,223797
1987-88,81828,48599,16055,10160,31563,45461,203993
1986-87,83020,48998,16264,10411,32040,46277,207338
1985-86,82161,49065,16576,9910,33645,47543,207875
1984-85,82008,49515,16122,10019,33694,47036,209041
1983-84,81150,49397,16041,10004,33755,48573,207668
1982-83,83853,48862,16778,10566,36038,48373,204658
1981-82,81987,47438,16107,10131,33435,49352,204776
1980-81,82010,48034,16964,10036,35354,47265,203886
1979-80,81065,46567,16993,9561,34166,43952,197239
"""
        |> fromCSV
        |> gather "StatType" "TRB" [ ( "TRB", "TRB" ), ( "AST", "AST" ), ( "STL", "STL" ), ( "BLK", "BLK" ), ( "TOV", "TOV" ), ( "PF", "PF" ), ( "PTS", "PTS" ) ]
```

```elm {m}
displaymvpTable : List String
displaymvpTable =
    mvpTable
        |> tableSummary 1000
```

```elm{l=hidden}

mvpTable : Table
mvpTable ="""
Season,MVP,POS
2021-22, N. Jokić,C
2020-21, N. Jokić,C
2019-20, G. Antetokounmpo,C
2018-19, G. Antetokounmpo,C
2017-18, J. Harden,SG
2016-17, R. Westbrook,PG
2015-16, S. Curry,PG
2014-15, S. Curry,PG
2013-14, K. Durant,SF
2012-13, L. James,SF
2011-12, L. James,SF
2010-11, D. Rose,PG
2009-10, L. James,SF
2008-09, L. James,SF
2007-08, K. Bryant,SG
2006-07, D. Nowitzki,PF
2005-06, S. Nash,PG
2004-05, S. Nash,PG
2003-04, K. Garnett,PF
2002-03, T. Duncan,PF
2001-02, T. Duncan,PF
2000-01, A. Iverson,PG
1999-00, S. O'Neal,C
1998-99, K. Malone,PF
1997-98, M. Jordan,SG
1996-97, K. Malone,PF
1995-96, M. Jordan,SG
1994-95, D. Robinson,PF
1993-94, H. Olajuwon,C
1992-93, C. Barkley,PF
1991-92, M. Jordan,SG
1990-91, M. Jordan,SG
1989-90, M. Johnson,PG
1988-89, M. Johnson,PG
1987-88, M. Jordan,SG
1986-87, M. Johnson,PG
1985-86, L. Bird,PF
1984-85, L. Bird,PF
1983-84, L. Bird,PF
1982-83, M. Malone,PF
1981-82, M. Malone,PF
1980-81, J. Erving,SF
1979-80, K. Abdul-Jabbar,C
"""
        |> fromCSV
```


```elm {m}
display3ptTable : List String
display3ptTable =
    threePointGraph
        |> tableSummary 43
```

```elm{l=hidden}

threePointGraph:Table
threePointGraph="""
Season,3P,3PA,3P%
2021-22,30598,86535,0.354
2020-21,27427,74822,0.367
2019-20,25862,72252,0.358
2018-19,27955,78742,0.355
2017-18,25807,71340,0.362
2016-17,23748,66422,0.358
2015-16,20953,59241,0.354
2014-15,19300,55137,0.350
2013-14,19054,52974,0.360
2012-13,17603,49067,0.359
2011-12,12693,36395,0.349
2010-11,15886,44313,0.358
2009-10,15822,44622,0.355
2008-09,16352,44583,0.367
2007-08,16124,44544,0.362
2006-07,14926,41672,0.358
2005-06,14086,39313,0.358
2004-05,13777,38748,.356
2003-04,12321,35492,0.347
2002-03,12200,34912,0.349
2001-02,12402,35074,0.354
2000-01,11524,32597,0.354
1999-00,11513,32614,.353
1998-99,6463,19080,0.339
1997-98,10450,30231,0.346
1996-97,14383,39943,0.360
1995-96,14000,38161,0.367
1994-95,12153,33889,0.359
1993-94,7301,21907,0.333
1992-93,6668,19824,0.336
1991-92,5587,16898,0.331
1990-91,5055,15812,0.320
1989-90,4829,14608,0.331
1988-89,4332,13431,0.323
1987-88,2979,9421,0.316
1986-87,2687,8913,0.301
1985-86,1774,6293,0.282
1984-85,1671,5917,0.282
1983-84,1120,4484,0.250
1982-83,1011,4248,0.238
1981-82,1129,4308,0.262
1980-81,936,3815,0.245
1979-80,1403,5003,0.280
"""
      |> fromCSV
```

add background pictures

```elm {m}
salaryTableDisplay : List String
salaryTableDisplay =
    salaryTable
        |> tableSummary 43
```

consider consequences when representing data

consider anonimity and showcasing personal data

iterative design would be useful

create your designs so you dont have to explain your data visualisation

position y 
pAggragate opCount

pie chartttttttt