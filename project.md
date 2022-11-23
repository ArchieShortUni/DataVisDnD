---
id: litvis
follows: data.md

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../css/datavis.less"

# Data Visualization Project Summary

{(whoami|} {|whoami)}
Archie Short archie.short@city.ac.uk

{(task|}

You should complete this datavis project summary document and submit it, along with any necessary supplementary files to **Moodle** by **Sunday 19th December, 5pm GMT**. Submissions will be awarded up to **80 marks** towards your coursework assessment total.

You are also encouraged to regularly commit and push changes to your datavis project throughout the term as you develop your project.

{|task)}

{(questions|}

- Do the spellcasters in dungeons and dragons 5th edition have stats that actually align with their narritive architypes
- Are the spellcasters spell-lists oriented around combat, roleplay or evenly between both
- Can a most versatile spellcaster be found from this data and if so which class is it
- Is there a difference in the distances spellcasters are effective in because of their spell-lists

{|questions)}

<div class="visualization"><h2>1. Visualization 1, school of magic radar diagrams</h2><span class="annotation"></span></fieldset></div>

![FullCasterRader](/Images/Full_casters_revised.png)
![HalfCasterRader](/Images/Half_casters_revised.png)
![PactCasterRader](/Images/Pact_casters_revised.png)

#### Bard Radar Example

All of the radar diagrams can be found in sketches.md

```elm{v}
radarBard : Spec
radarBard =
    let
        table =
            V.dataFromColumns "table" []
                << V.dataColumn "key"(V.vStrs ["Conjuration","Divination","Illusion",  "Necromancy","Enchantment", "Transmutation", "Abjuration", "Evocation"])


                << V.dataColumn "value" (V.vNums [8,18,20,8,34,21,11,15])
                << V.dataColumn "category" (V.vStrs ["Cleric","Cleric","Cleric","Cleric","Cleric","Cleric","Cleric","Cleric"])


        ds =
            V.dataSource
                [ table []
                , V.data "keys" [ V.daSource "table" ]
                    |> V.transform [ V.trAggregate [ V.agGroupBy [ V.field "key" ] ] ]
                ]

        si =
            V.signals
                << V.signal "radius" [ V.siUpdate "width / 2" ]

        sc =
            V.scales
                << V.scale "angular"
                    [ V.scType V.scPoint
                    , V.scDomain (V.doData [ V.daDataset "table", V.daField (V.field "key") ])
                    , V.scRange (V.raNums [ -pi, pi ])
                    , V.scPadding (V.num 0.5)
                    ]
                << V.scale "radial"
                    [ V.scType V.scLinear
                    , V.scDomain (V.doData [ V.daDataset "table", V.daField (V.field "value") ])
                    , V.scDomainMin (V.num 0)
                    , V.scRange (V.raSignal "[0, radius]")
                    , V.scZero V.true
                    , V.scNice V.niFalse
                    ]
                << V.scale "cScale"
                    [ V.scType V.scOrdinal
                    , V.scDomain (V.doData [ V.daDataset "table", V.daField (V.field "category") ])
                    , V.scRange (V.raScheme (V.str "category10") [])
                    ]

        en =
            V.encode [ V.enEnter [ V.maX [ V.vSignal "radius" ], V.maY [ V.vSignal "radius" ] ] ]

        nestedMk =
            V.marks
                << V.mark V.line
                    [ V.mName "category-line"
                    , V.mFrom [ V.srData (V.str "facet") ]
                    , V.mEncode
                        [ V.enEnter
                            [ V.maInterpolate [ V.vStr "linear-closed" ]
                            , V.maX [ V.vSignal "scale('radial', datum.value) * cos(scale('angular', datum.key))" ]
                            , V.maY [ V.vSignal "scale('radial', datum.value) * sin(scale('angular', datum.key))" ]
                            , V.maStroke [ V.vStr "rgb(171,109,172)"]
                            , V.maStrokeWidth [ V.vNum 2 ]
                            , V.maFill [ V.vStr "rgb(171,109,172)" ]
                            , V.maFillOpacity [ V.vNum 0.4 ]
                            ]
                        ]
                    ]
        mk =
            V.marks
                << V.mark V.group
                    [ V.mName "categories"
                    , V.mZIndex (V.num 1)
                    , V.mFrom [ V.srFacet (V.str "table") "facet" [ V.faGroupBy [ V.field "category" ] ] ]
                    , V.mGroup [ nestedMk [] ]
                    ]
                << V.mark V.rule
                    [ V.mName "radial-grid"
                    , V.mFrom [ V.srData (V.str "keys") ]
                    , V.mZIndex (V.num 0)
                    , V.mEncode
                        [ V.enEnter
                            [ V.maX [ V.vNum 0 ]
                            , V.maY [ V.vNum 0 ]
                            , V.maX2 [ V.vSignal "radius * cos(scale('angular', datum.key))" ]
                            , V.maY2 [V.vSignal "radius * sin(scale('angular', datum.key))" ]
                            , V.maStroke [ V.vStr "lightGrey" ]
                            , V.maStrokeWidth [ V.vNum 1 ]
                            ]
                        ]
                    ]
                << V.mark V.text
                    [ V.mName "key-label"
                    , V.mFrom [ V.srData (V.str "keys") ]
                    , V.mEncode
                        [ V.enEnter
                            [ V.maX [ V.vSignal "(radius + 5) * cos(scale('angular', datum.key))" ]
                            , V.maY [ V.vSignal "(radius + 5) * sin(scale('angular', datum.key))" ]
                            , V.maText [ V.vField (V.field "key") ]
                            , V.maAlign
                                [ V.ifElse "abs(scale('angular', datum.key)) > PI / 2"
                                    [ V.vStr "right" ]
                                    [ V.vStr "left" ]
                                ]
                            , V.maBaseline
                                [ V.ifElse "scale('angular', datum.key) > 0"
                                    [ V.vStr "top" ]
                                    [ V.ifElse "scale('angular', datum.key) == 0"
                                        [ V.vStr "middle" ]
                                        [ V.vStr "bottom" ]
                                    ]
                                ]
                            , V.maFill [ V.black ]
                            , V.maFontSize [V.vNum 12]

                            , V.maFontWeight [ V.vStr "bold" ]
                            ]
                        ]
                    ]

                ---Radar marking outline and 20 % lines
                 << V.mark V.line
                    [ V.mName "twenty-line"
                    , V.mFrom [ V.srData (V.str "keys") ]
                    ,V.mEncode
                        [ V.enEnter
                            [ V.maInterpolate [ V.vStr "linear-closed" ]
                            , V.maX [ V.vSignal "0.2 * radius * cos(scale('angular', datum.key))" ]
                            , V.maY [ V.vSignal "0.2 * radius * sin(scale('angular', datum.key))" ]
                            , V.maStroke [ V.vStr "lightGrey" ]
                            , V.maStrokeWidth [ V.vNum 1 ]
                            ]
                        ]
                    ]

                << V.mark V.line
                    [ V.mName "fouty-line"
                    , V.mFrom [ V.srData (V.str "keys") ]
                    , V.mEncode
                        [ V.enEnter
                            [ V.maInterpolate [ V.vStr "linear-closed" ]
                            , V.maX [ V.vSignal "0.4 * radius * cos(scale('angular', datum.key))" ]
                            , V.maY [ V.vSignal "0.4 * radius * sin(scale('angular', datum.key))" ]
                            , V.maStroke [ V.vStr "lightGrey" ]
                            , V.maStrokeWidth [ V.vNum 1 ]
                            ]
                        ]
                    ]

                << V.mark V.line
                    [ V.mName "sixty-line"
                    , V.mFrom [ V.srData (V.str "keys") ]
                    , V.mEncode
                        [ V.enEnter
                            [ V.maInterpolate [ V.vStr "linear-closed" ]
                            , V.maX [ V.vSignal "0.6 * radius * cos(scale('angular', datum.key))" ]
                            , V.maY [ V.vSignal "0.6 * radius * sin(scale('angular', datum.key))" ]
                            , V.maStroke [ V.vStr "lightGrey" ]
                            , V.maStrokeWidth [ V.vNum 1 ]
                            ]
                        ]
                    ]


                << V.mark V.line
                    [ V.mName "eighty-line"
                    , V.mFrom [ V.srData (V.str "keys") ]
                    , V.mEncode
                        [ V.enEnter
                            [ V.maInterpolate [ V.vStr "linear-closed" ]
                            , V.maX [ V.vSignal "0.8 * radius * cos(scale('angular', datum.key))" ]
                            , V.maY [ V.vSignal "0.8 * radius * sin(scale('angular', datum.key))" ]
                            , V.maStroke [ V.vStr "lightGrey" ]
                            , V.maStrokeWidth [ V.vNum 1 ]
                            ]
                        ]
                    ]

                << V.mark V.line
                    [ V.mName "outer-line"
                    , V.mFrom [ V.srData (V.str "radial-grid") ]
                    , V.mEncode
                        [ V.enEnter
                            [ V.maInterpolate [ V.vStr "linear-closed" ]
                            , V.maX [ V.vField (V.field "x2") ]
                            , V.maY [ V.vField (V.field "y2") ]
                            , V.maStroke [ V.vStr "lightGrey" ]
                            ,V.maStrokeWidth [ V.vNum 1 ]
                            ]
                        ]
                    ]
    in
    V.toVega
        [ V.width 400, V.height 400, V.padding 80,V.autosize [ V.asNone, V.asPadding ], ds, si [], sc [], en, mk [] ]


```

```elm {l=hidden}
spellData : Data
spellData =
    (dataFromColumns []
      << dataColumn "school"(strColumn "school" classesPerSpell |> strs)
      << dataColumn "level"(strColumn "level" classesPerSpell |> strs)
      << dataColumn "Type"(strColumn "Type" classesPerSpell |> strs)
      << dataColumn "Cleric Default Spell Type"(strColumn "Type" classesPerSpell |>strs )
      << dataColumn "Class"(strColumn "Class" classesPerSpell |> strs)
      << dataColumn "Range"(strColumn "Range" classesPerSpell |> strs)
    )
      []
```

```elm {l=hidden}
auraData : Data
auraData =
     (dataFromColumns []
      << dataColumn "rangeType"(strColumn "rangeType" aura |> strs)
      << dataColumn "Class"(strColumn "Class" aura |> strs)
      << dataColumn "auraRange"(numColumn "Range" aura |> nums)
    )
      []
```

```elm {l=hidden}
touchData : Data
touchData =
 (dataFromColumns []
      << dataColumn "rangeType"(strColumn "rangeType" selfSpells |> strs)
      << dataColumn "ClassT"(strColumn "Class" selfSpells |> strs)
      << dataColumn "touchRange"(numColumn "Range" selfSpells |> nums)
      << dataColumn "rangeBool"(strColumn "rangeBool" selfSpells |> strs)


    )
      []
```

<div class="visualization"><h2>1. Visualization 2, spell range dashboard</h2><span class="annotation"></span></fieldset></div>

```elm {v}
rangeDashboard : Spec
rangeDashboard =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])


        classColours =
            -- Create a mapping from party to its (rgb) colour
            categoricalDomainMap
                [ ( "bard", "rgb(180,141,178)" )
                , ( "cleric", "rgb(138,140,140)" )
                , ( "druid", "rgb(139,148,91)" )
                , ( "paladin", "rgb(196,162,57)" )
                , ( "ranger", "rgb(77,124,95)" )
                , ( "sorcerer", "rgb(190,83,86)" )
                , ( "warlock", "rgb(125,57,168)" )
                , ( "wizard", "rgb(68,117,205)" )
                ,("","rgb(255,255,255)")
                 ,( "Target Touch", "rgb(147,12,16)" )
                , ( "Target Self", "rgb(99, 26, 20)" )
                ]

        trans =
            transform
                << calculateAs """if(datum.rangeBool == 'TRUE','Target Touch',
                                    if(datum.rangeBool == 'FALSE','Target Self',2))""" "rangeT"
                << lookup "Class" auraData "Class"(luFields["auraRange"])



        encBoxPlot =
            encoding
                << position X [ pName "Range",pTitle "Spell Range (ft)", pQuant, pScale [ scZero False ] ]
                << position Y [ pName "Class", pAxis [ axTitle "" ,axDomain False,axTicks False] ]
                << color [mName "Class", mScale classColours,mTitle"Legend"]

        spec_box =
            asSpec[width 400,spellData, encBoxPlot [], boxplot [maExtent exRange]]

        enc =
            encoding
                << position X
                    [ pName "rangeT"
                    , pAggregate opCount -- Find total in each group
                    , pAxis
                        [ axValues (nums [ 0.5 ]) -- Show 50% marker only
                        , axTitle "Self : Touch Spell Ratio" -- No axis title
                        , axFormat "%" -- Format axis label numbers as a percentages
                        , axZIndex 1 -- Place axis grid line on top
                        , axDomain False -- No axis baseline
                        ]
                    , pStack stNormalize -- Normalise width of all bars
                    ]
                << position Y
                    [ pName "ClassT"
                    , pAxis [ axDomain False, axTitle "" ,axTicks False]

                    ]
                << color [ mName "rangeT"
                    , mScale classColours
                    ]

        spec_normal =
            asSpec[enc[],bar []]




    in
    toVegaLite [ width 400, cfg[],trans[],touchData, hConcat [ spec_box, spec_normal ]]
```

```elm {v interactive}
auraCircles : Spec
auraCircles =
    let
      cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

      classColours =
            -- Create a mapping from party to its (rgb) colour
            categoricalDomainMap
                [ ( "bard", "rgb(180,141,178)" )
                , ( "cleric", "rgb(138,140,140)" )
                , ( "druid", "rgb(139,148,91)" )
                , ( "paladin", "rgb(196,162,57)" )
                , ( "ranger", "rgb(77,124,95)" )
                , ( "sorcerer", "rgb(190,83,86)" )
                , ( "warlock", "rgb(125,57,168)" )
                , ( "wizard", "rgb(68,117,205)" )
                ]
      trans =
            transform
                << lookup "Class" auraData "Class"(luFields["auraRange"])

      encCircle =
        encoding
            << position X [pName "Class" ,pAxis[axTitle"Average Class Aura Range (ft)",axOffset -35,axLabelAngle 0]]
            << color [mName "Class", mScale classColours,mLegend[]]

            << position Y [pName "Range" ,pAggregate opMin, pAxis [axTitle "",axTicks False,axOffset 60,axDomain False]]
            << size
                    [ mName "auraRange"
                    , mAggregate opMean
                    , mScale [ scRange (raMax 5000) ] -- Size of circles
                    ,mLegend[]
                    ]
            << tooltip [ tName "auraRange"  ]

      specCircle =
            asSpec[encCircle[], circle [ maFillOpacity 0.8, maStroke "black", maStrokeOpacity 0.3, maStrokeWidth 1 ]]

    in
    toVegaLite [cfg[] ,width 750,  spellData,trans[],layer [ specCircle ]]
```

<div class="visualization"><h2>1. Visualization 3, damage, utility and healing</h2><span class="annotation"></span></fieldset></div>

```elm {v interactive}
stackedBar1 : Spec
stackedBar1 =
    let

        typeColours =
            -- Create a mapping from party to its (rgb) colour
            categoricalDomainMap
                [ ("Damage","rgb(99, 26, 20)")
                ,("Combat Utility","rgb(147,12,16)")
                ,("Utility","rgb(124, 140, 91)")
                ,("Healing"," 	rgb(234, 237, 152)")
                ]

        enc =
            encoding
                << position X
                    [ pName "Type"
                    , pAggregate opCount
                    , pTitle "Number of Spells"
                    ,pOrdinal


                    ]
                << position Y
                    [ pName "Class"
                    , pSort [ soAscending ]
                    , pAxis [axTitle"", axTicks False, axDomain False, axOffset 5 ]
                    ]
                << color
                    [mCondition (prParam "select")
                        [ mName "Type", mScale typeColours, mTitle "Spell Type" ]
                        [ mStr "black" ]

                    ]
                << opacity
                    [ mCondition (prParam "select")
                        [ mNum 1 ]
                        [ mNum 0.1 ]
                    ]
                << order [oName "Type",oAggregate opCount,oSort [ soDescending ]]
                << tooltips[[tName "Type",tTitle"Spells",tAggregate opCount],[tName"Type",tTitle "Spell Type"]]

        ps =
            params
             ---   << param "mySelection"
                 ---   [ paSelect sePoint [ seFields [ "Type" ] ]
                  ---  , paBindLegend "click"
                   --- ]

                << param "highlight" [ paSelect sePoint [ seOn "mouseover" ] ]
                << param "select" [ paSelect sePoint [seFields [ "Type" ]] ]
    in
    toVegaLite [ width 700,spellData,ps [], enc [], bar [] ]
```

{(insights|}

###Insight 1: Narrative Accuracy
This insight is related to question 1, I believe that in the most part yes, the spellcasters do have a kit that match their narrative description and this can be shown by putting together a few pairs of the radar diagrams.

Druid and Ranger are both classes that are heavily based in nature, they both have a duty of protecting it and keeping balance and that theme can be seen by their similar shape across their charts. In terms of the schools of magic the ones that can be closest linked to nature are Divination, Conjuration and Transmutation. Both the Ranger and Druid make heavy use of these in their spell lists showing that these narrative ideas such as a connection to nature can be seen through their spell choice. It also shows that really the only difference between the two in terms of spellcasting is that the rangers spells lean more towards protection with Abjuration while the druids lean more towards offence with Evocation.

Rather than going through all the classes I’ll make another comparison, Paladin and Cleric are both powered through what is called Divine magic. Unlike nature where I know what schools make up that phenomenon, we could potentially reverse the thinking here. It’s a similar situation where the two classes share a similar ratio across three schools of magic; Divination, Abjuration and Evocation. If we follow the same logic, we could now hypothesize that these are the three schools that make up divine magic but that I am less sure on.

###Insight 2: Roleplay VS Combat

This is in relation to question 2, it seems to differ from class to class but in general they are generally very combat oriented, while most classes have more utility spells than straight damage when you add together combat utility and damage the only classes where utility is greater is Bard and Cleric.
I would still say though that Bard is the only true roleplay focused class because of the Clerics focus on healing, which is still combat related. Even though the amount of healing spells they have is smaller than other categories even few make a large difference compared to utility or damage.

###Insight 3: Versitility

This insight relates to questions 3 and 4. Versatility can be measured across all the different visualisations here, but I think the most important one to start with is the radar charts. The Warlock in my opinion has the strongest start as their chart spreads out in almost every direction at least a little bit showing they have some options for every situation.
Looking at the range dashboard the Warlock doesn’t have the furthest range compared to other spellcasters however what they do have is their spells spread evenly across that 0 to 60 feet range as shown in the boxplot.
Looking at the third visualisation the Warlock also has one of the most even splits between utility, combat utility and damage. Although they don’t have any healing spells that’s a category you only need one person in the party to cover so in general when creating a character you don’t have to worry about it.

#### Insight side note

I would not count it as one of my 3 insights as it had nothing to do with my questions, but I would say the discovery of what I believe to be the correct order for the magic school circle through this data is probably the best thing to come out of my visualisations, I explain in my design justification how I came to the layout. In terms of narrative, the shape of the classes’ radar diagrams symbolises a kind of balance. The fact that each class has a unique shape and identity that you can make conclusions from is quite useful for future content creation.

Dungeons and dragons is a game based on community creation, people make their own classes, subclasses and lore. I believe that this could be used as another tool to measure the effectiveness of someone’s own creations and help workshop new spellcaster types as their spell list is being put together.

{|insights)}

{(designJustification|}

### Justification 1: Radar Charts

According to _(Munzner, 2015, pp.94–115)_ the use of spatial region is the best channel to show categorical attributes so this I knew use of this was a good way of showing the different schools of magic used for each class. In total there are 8 schools of magic and with 8 classes representing them with any kind of bar chart would have at least 64 bars which makes comparison difficult. You could use stacked bars but they would be stacked so densely it would be hard to make conclusions without use of a tooltip. Trying to fit all the distinct data in an ordinary bar graph like that would also make assembly and estimation, two of the three perceptual activities _(Cleveland, 1994)_ associated with data visualisation more difficult.
For these reasons I decided to go with radar charts, the number of different categories that radar charts are ideal for are 3-8 _(www.ifm.eng.cam.ac.uk, n.d.)_ so the 8 schools of magic just about fit. They shape created by the data can give a clear summary _(www.pluscharts.com, 2019)_ and I thought when it comes to the classes their different shapes would help answer my questions. A unique feature of the categories I have chosen to work with here is they can be put together to make a magic circle. Order of the categories is extremely important in radar diagrams so I realised I could use the order of a magic circle to make the most out of the data I’m using. Looking around there didn’t seem to be a canonically correct version rather just many widely accepted versions. I chose one I thought would work and tried it out _(FishDunce, 2018)_. For the most part it worked apart from half the classes had their shape disrupted between the Illusion and divination categories (this can be seen in the radar1.docx file). I swapped around these two categories, and it made the shapes more cohesive.
While it isn’t a huge change or an explicit feature or trend in the data itself, displaying it in this way has now given what I believe to be the definitive school of magic circle and while it doesn’t relate to any of the questions I was asking with the dataset it is an unexpected insight only gained through the iteration of this design choice.

### Justification 2: Layout

I would say my layout choices can be split into three different parts, the layout of radars, the range information dashboard and then the decision to use horizontal bars instead of vertical ones which I think would fall under layout as well.

Starting with the layout of my radar chart diagrams I thought juxtaposition was the best option, adding more than 3 groups onto a radar chart makes it very visually confusing and hard to compare _(Effective Use of Radar Charts)_. To avoid this when having quite a few groups it is a good practice to use small multiples which avoid a cluttered figure _(Holtz)_. Therefore, I went for a 3 in a row layout instead of having them all in a column or more tightly packed layout. It also allowed for a sloping pyramid like motion to the set of charts as you go down which is quite visually appealing. Not only is there juxtaposition between the classes in the layout but also between casting type, it is categorised between full casters, half casters and pact caster. I wasn’t sure how different the data would look between these categories but there are mechanical differences within these groups in the game so decided it was best not to mix them all up as they are known to be separate. This way if there were comparisons they could still be made. There are other ways of categorising the spellcasters such as divine vs arcane and countless others but I decided not too make too many distinctions at it might get too confusing.

I couldn’t quite get the radars laid out how I wanted using Vega code so Instead I compiled them in an image for how I would have wanted them to look, I had no planned interactivity but even if I did I would have had to sacrifice it for the layout I wanted to get with my current Vega knowledge.

The next layout design choice was to do with the visualisations to do with range. My layout choice here stems from the fact I wanted to represent the range data in different visualisations rather than all in one. In sketches.md you can see I tried to compile average range with average aura into one visual however I realised I was losing a lot of detail in the data by trying to compile it all into one and thought a dashboard of all the different range data would be better. I decided this way was better because it would allow me to use every bit of the range data in some way even if it didn’t turn out to be as useful as I thought it would be like the touch to self spell range ratio visualisation where it’s all extremely similar. Compared to what dashboards can be _(Sarikaya et al.)_ it is very basic as it doesn’t update in real time or allow for any customisation, but I think that is a limit of the dataset just as much as it is a limit of my design.

The final part of layout I want to talk about is the choice of using horizontal bars over vertical bars, in a blog by Ann K. Emery _(Emery, 2017)_ they explain you should use horizontal bars for nominal and categorical data, all the different charts are defined by the class category which is why I went for this choice. They also mention how this way you can bring the most important data or any data you want to bring focus to straight to the top however I decided to keep the order the classes are presented consistent throughout so you can jump between visualisations and know where to look decreasing the cognitive load allowing the reader to only need to think about the actual comparisons.

### Justification 3: Colour

My colour choice is widely based on existing colour schemes already associated with dungeons and dragons, for each class I assigned it a colour widely used by the community to represent it and then kept that use consistent throughout the visualisations where it applied. As this scheme of colours isn’t one generally used in data visualisation, I check on a colourblind simulator _(Colblindor, 2016)_ to make sure they worked and kept labels so even in the worst-case scenario you could navigate based on text.

Even though I kept the labels I wanted members of the dungeons and dragons community to be able to look at the visualisations and immediately be able to recognise what is going on so made sure to they would be recognisable. I also believe this approach means that for example someone could be scrolling through a website and if they’re a fan of dungeons and dragons it would catch their eye enough to get them to stop and take a look.

For the visualisations that split bars into multiple sections such as the touch to self ratio chart and the spell type chart I had to find an appropriate colour scheme. Rather than generating one online until I was satisfied, I read through “your friendly guide to colours in data visualization” _(Muth, 2018)_ and decided to “copy from masters”. In this case the source I would be taking inspiration from is official dungeons and dragons’ books and guide, they have a tried and tested set of colours that across their content which is recognisable to the community which is what I was striving for.

{|designJustification)}

{(references|}

[1] Munzner, T. (2015). Visualization analysis & design. Boca Raton: Crc Press, Taylor & Francis Group, pp.94–115.

[2] Cleveland, W.S. (1994). The elements of graphing data. Murray Hill, N.J.: At & T Bell Laboratories.

[3] www.ifm.eng.cam.ac.uk. (n.d.). Polar charts, Radar charts. [online] Available at: https://www.ifm.eng.cam.ac.uk/research/dstools/polar-charts-radar-charts/

[4] www.pluscharts.com. (2019) Why and when to use a Spider and Radar Chart? [online] Available at: https://www.pluscharts.com/why-and-when-to-use-spider-and-radar-chart/

[5] FishDunce (2018). [OC] Schools of Magic Periodic Circle. [online] Reddit. Available at: https://www.reddit.com/r/DnD/comments/9uspaa/oc_schools_of_magic_periodic_circle/

[6] Effective Use of Radar Charts. (n.d.). [online] Available at: https://msktc.org/lib/docs/KT_Toolkit/Charts_and_Graphs/Charts_and_Graphics_Radar_508c.pdf.
Holtz, Y. (n.d.). The Radar chart and its caveats. [online] www.data-to-viz.com. Available at: https://www.data-to-viz.com/caveat/spider.html.

[7] Sarikaya, A., Correll, M., Bartram, L., Tory, M. and Fisher, D. (n.d.). What Do We Talk About When We Talk About Dashboards? [online] pp.3–6. Available at: https://research.tableau.com/sites/default/files/DashboardsConspiracy_final.pdf.

[8] Emery, A.K. (2017). Depict Data Studio. [online] Depict Data Studio. Available at: https://depictdatastudio.com/when-to-use-horizontal-bar-charts-vs-vertical-column-charts/.

[9] Colblindor (2016). Coblis — Color Blindness Simulator – Colblindor. [online] Color-blindness.com. Available at: https://www.color-blindness.com/coblis-color-blindness-simulator/.

[10] Muth, L.C. (2018). Your Friendly Guide to Colors in Data Visualisation. [online] blog.datawrapper.de. Available at: https://blog.datawrapper.de/colorguide/#Copy-from-the-masters.

{|references)}

```

```
