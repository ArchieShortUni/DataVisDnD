---
id: litvis
follows: data.md

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../css/datavis.less"

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

goldenRatio : Float
goldenRatio =
    1.618
```

```elm {l=hidden}
cantripData : Data
cantripData =
    (dataFromColumns []

      << dataColumn "Level"(numColumn "Level" cantripTable |> nums)
      << dataColumn "Cantrips"(numColumn "Cantrips" cantripTable |> nums)
      << dataColumn "ClassCantrips"(strColumn "Class" cantripTable |> strs)
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

```elm {l v}
cantripsPerLevel: Spec
cantripsPerLevel =
    let
      enc =
       encoding
         << position X [ pName "Level", pQuant, pTitle "" ]
         << position Y [ pName "Cantrips", pQuant ]
         << color [ mName "ClassCantrips", mTitle "Class" ]


    in
    toVegaLite [ width 400, cantripData, enc [], line [] ]
```

```elm { v interactive highlight=13}
testBar2 : Spec
testBar2 =
    let
      cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

      classColours =
            -- Create a mapping from party to its (rgb) colour
            categoricalDomainMap
                [ ( "bard", "rgb(171,109,172)" )
                , ( "cleric", "rgb(145,161,178)" )
                , ( "druid", "rgb(122,133,59)" )
                , ( "paladin", "rgb(181,158,84)" )
                , ( "ranger", "rgb(80,127,98)" )
                , ( "sorcerer", "rgb(153,46,46)" )
                , ( "warlock", "rgb(123,70,155)" )
                , ( "wizard", "rgb(42,80,161)" )
                ]

      enc =
        encoding
          << position X [pName "Class" ,pAxis[]]
          << position Y [pName "Type" ,pAggregate opCount,pTitle "Spells"]
          << column [fName "Type", fSpacing 10]
          << color [mName "Class", mScale classColours]
    in
    toVegaLite [cfg[],width 130, height 300, spellData, enc[],bar[ maTooltip ttEncoding ]]

```

```elm {v l interactive}
stackedBar : Spec
stackedBar =
    let

        typeColours =
            -- Create a mapping from party to its (rgb) colour
            categoricalDomainMap
                [ ("Damage","rgb(153,46,46)")
                ,("Combat Utility","rgb(123,70,155)")
                ,("Utility","rgb(42,80,161)")
                ,("Healing","rgb(145,161,178)")
                ]

        enc =
            encoding
                << position X
                    [ pName "Type"
                    , pAggregate opCount
                    , pTitle "Spells"
                    ,pOrdinal
                    ,pSort[soCustom (strs ["Damage","Combat Utility","Utility","Healing"])]

                    ]
                << position Y
                    [ pName "Class"
                    , pSort [ soDescending ]
                    , pAxis [ axTicks False, axDomain False, axOffset 5 ]
                    ]
                << color
                    [ mCondition (prParam "mySelection")
                        [ mName "Type", mScale typeColours, mTitle "" ]
                        [ mStr "black" ]
                    ]
                << opacity
                    [ mCondition (prParam "mySelection")
                        [ mNum 1 ]
                        [ mNum 0.1 ]
                    ]
                << tooltips[[tName "Type",tTitle"Spells",tAggregate opCount]]

        ps =
            params
                << param "mySelection"
                    [ paSelect sePoint [ seFields [ "Type" ] ]
                    , paBindLegend "click"
                    ]
    in
    toVegaLite [ width 700,spellData,ps [], enc [], bar [] ]
```

```elm{ v interactive highlight=13}
testBar6 : Spec
testBar6 =
    let
      cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

      classColours =
            -- Create a mapping from party to its (rgb) colour
            categoricalDomainMap
                [ ( "bard", "rgb(171,109,172)" )
                , ( "cleric", "rgb(145,161,178)" )
                , ( "druid", "rgb(122,133,59)" )
                , ( "paladin", "rgb(181,158,84)" )
                , ( "ranger", "rgb(80,127,98)" )
                , ( "sorcerer", "rgb(153,46,46)" )
                , ( "warlock", "rgb(123,70,155)" )
                , ( "wizard", "rgb(42,80,161)" )
                ]

      enc =
        encoding
          << position X [pName "Class" ,pAxis[]]
          << position Y [pName "Range" ,pAggregate opCount,pTitle "Range"]
          << color [mName "Class", mScale classColours]
    in
    toVegaLite [cfg[],width 130, height 300, spellData, enc[],bar[ maTooltip ttEncoding ]]

```

```elm {v }
categoricalScatter : Spec
categoricalScatter =
    let
        data =
            dataFromUrl ("https://cdn.jsdelivr.net/npm/vega-datasets@2.2/data/disasters.csv") []

        trans =
            -- Filter out the total from the categories shown
            transform
                << filter (fiExpr "datum.Entity !== 'All natural disasters'")

        enc =
            encoding
                << position X [ pName "Year", pTemporal, pAxis [ axTitle "", axGrid False ] ]
                << position Y [ pName "Entity", pTitle "" ]
                << size
                    [ mName "Deaths"
                    , mQuant
                    , mLegend [ leTitle "Annual Global Deaths", leClipHeight 30 ]
                    , mScale [ scRange (raMax 5000) ] -- Size of circles
                    ]
                << color [ mName "Entity", mLegend [] ]
    in
    toVegaLite
        [ width 540
        , height 350
        , data
        , trans []
        , enc []
        , circle [ maFillOpacity 0.8, maStroke "black", maStrokeOpacity 0.3, maStrokeWidth 1 ]
        ]
```

Make the cirle size dependant on aura average for each class

```elm {v}
testBar3 : Spec
testBar3 =
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
              ---  << filter (fiExpr "datum.Range < 200 ")
                << filter (fiLessThan "Range" (num 200))
                << lookup "Class" auraData "Class"(luFields["auraRange"])


      enc =
        encoding
          << position X [pName "Class" ,pAxis[]]
          << color [mName "Class", mScale classColours]

      encBar =
        encoding
          << position Y [pName "Range" ,pAggregate opMean,pTitle "Average Range"]
          << column [fName "Class", fSpacing 10]

      specBar =
            asSpec [ encBar [], bar [ maSize 5] ]

      encCircle =
        encoding
            << position Y [pName "Range" ,pAggregate opMin]
            << size
                    [ mName "auraRange"
                    , mAggregate opMean
                    , mScale [ scRange (raMax 5000) ] -- Size of circles
                    ]
      specCircle =
            asSpec[encCircle[], circle [ maFillOpacity 0.8, maStroke "black", maStrokeOpacity 0.3, maStrokeWidth 1 ]]

    in
    toVegaLite [cfg[],width 650, height 300, spellData,trans[], enc[],layer [ specBar,specCircle ]]
```

```elm {h = hidden v }
tukeyBoxplots : Spec
tukeyBoxplots =
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


               --- << filter (fiExpr "datum.Range < 900 ")
        enc =
            encoding
                << position X [ pName "Range",pTitle "Spell Range", pQuant, pScale [ scZero False ] ]
                << position Y [ pName "Class", pAxis [ axLabelAngle  10, axTitle "" ,axDomain False] ]
                << color [mName "Class", mScale classColours]

    in
    toVegaLite [ width 400, cfg[],spellData, enc [], boxplot [maExtent exRange] ]
```

```elm { v}
popNormalised : Spec
popNormalised =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        classColours =
            -- Create a mapping from party to its (rgb) colour
            categoricalDomainMap
                [ ( "Touch", "rgb(190,83,86)" )
                , ( "Self", "rgb(44,37,36)" )

                ]


        trans =
            transform
                << calculateAs """if(datum.rangeBool == 'TRUE','Touch',
                                    if(datum.rangeBool == 'FALSE','Self',2))""" "rangeT"
        enc =
            encoding
                << position X
                    [ pName "rangeT"
                    , pAggregate opCount -- Find total in each group
                    , pAxis
                        [ axValues (nums [ 0.5 ]) -- Show 50% marker only
                        , axTitle "" -- No axis title
                        , axFormat "%" -- Format axis label numbers as a percentages
                        , axZIndex 1 -- Place axis grid line on top
                        , axDomain False -- No axis baseline
                        ]
                    , pStack stNormalize -- Normalise width of all bars
                    ]
                << position Y
                    [ pName "ClassT"
                    , pAxis [ axDomain False,axLabelAngle  10, axTitle "" ]

                    ]
                << color [ mName "rangeT"
                    , mScale classColours,mTitle "0 Range Spells"
                    ]
    in
    toVegaLite [ width 400, cfg[],trans[],touchData, enc [], bar [] ]
```

Combine premise of both ideas into one image, turkey plot much better for spell

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
                ,( "Touch", "rgb(190,83,86)" )
                , ( "Self", "rgb(44,37,36)" )
                ]

        trans =
            transform
                << calculateAs """if(datum.rangeBool == 'TRUE','Touch',
                                    if(datum.rangeBool == 'FALSE','Self',2))""" "rangeT"
                << lookup "Class" auraData "Class"(luFields["auraRange"])



        encBoxPlot =
            encoding
                << position X [ pName "Range",pTitle "Spell Range", pQuant, pScale [ scZero False ] ]
                << position Y [ pName "Class", pAxis [ axTitle "" ,axDomain False] ]
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
                    , pAxis [ axDomain False, axTitle "" ]

                    ]
                << color [ mName "rangeT"
                    , mScale classColours
                    ]

        spec_normal =
            asSpec[enc[],bar []]




    in
    toVegaLite [ width 400, cfg[],trans[],touchData, hConcat [ spec_box, spec_normal ]]
```

```elm {v}
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
            << position X [pName "Class" ,pAxis[axTitle"Average Class Aura Range",axOffset -35,axLabelAngle 0]]
            << color [mName "Class", mScale classColours,mLegend[]]

            << position Y [pName "Range" ,pAggregate opMin, pAxis [axTitle "",axTicks False,axOffset 60,axDomain False]]
            << size
                    [ mName "auraRange"
                    , mAggregate opMean
                    , mScale [ scRange (raMax 5000) ] -- Size of circles
                    ,mLegend[]
                    ]
      specCircle =
            asSpec[encCircle[], circle [ maFillOpacity 0.8, maStroke "black", maStrokeOpacity 0.3, maStrokeWidth 1 ]]

    in
    toVegaLite [cfg[] ,width 750,  spellData,trans[],layer [ specCircle ]]
```

#### Bard

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

#### Cleric

```elm{v}
radarCleric : Spec
radarCleric =
    let
        table =
            V.dataFromColumns "table" []
                << V.dataColumn "key"(V.vStrs ["Conjuration","Divination","Illusion",  "Necromancy","Enchantment", "Transmutation", "Abjuration", "Evocation"])


                << V.dataColumn "value" (V.vNums [13,17,1,17,6,11,26,22])
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
                            , V.maStroke [ V.vStr "rgb(138,140,140)"]
                            , V.maStrokeWidth [ V.vNum 2 ]
                            , V.maFill [ V.vStr "rgb(138,140,140)" ]
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

#### Druid

```elm{v}
radarDruid : Spec
radarDruid =
    let
        table =
            V.dataFromColumns "table" []
                << V.dataColumn "key"(V.vStrs ["Conjuration","Divination","Illusion",  "Necromancy","Enchantment", "Transmutation", "Abjuration", "Evocation"])


                << V.dataColumn "value" (V.vNums [33,14,2,4,11,50,15,26])
                << V.dataColumn "category" (V.vStrs ["Druid","Druid","Druid","Druid","Druid","Druid","Druid","Druid"])


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
                            , V.maStroke [ V.vStr "rgb(122,133,59)"]
                            , V.maStrokeWidth [ V.vNum 2 ]
                            , V.maFill [ V.vStr "rgb(122,133,59)" ]
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

#### Paladin

```elm{v}
radarPaladin : Spec
radarPaladin =
    let
        table =
            V.dataFromColumns "table" []
                << V.dataColumn "key"(V.vStrs ["Conjuration", "Divination","Illusion", "Necromancy","Enchantment", "Transmutation", "Abjuration", "Evocation"])


                << V.dataColumn "value" (V.vNums [4,5,0,1,7,3,17,12])
                << V.dataColumn "category" (V.vStrs ["Paladin","Paladin","Paladin","Paladin","Paladin","Paladin","Paladin","Paladin"])


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
                            , V.maStroke [ V.vStr "rgb(181,158,84)"]
                            , V.maStrokeWidth [ V.vNum 2 ]
                            , V.maFill [ V.vStr "rgb(181,158,84)" ]
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

#### Ranger

```elm{v}
radarRanger : Spec
radarRanger =
    let
        table =
            V.dataFromColumns "table" []
                << V.dataColumn "key"(V.vStrs ["Conjuration", "Divination","Illusion", "Necromancy","Enchantment", "Transmutation", "Abjuration", "Evocation"])


                << V.dataColumn "value" (V.vNums [11,11,1,0,2,16,10,4])
                << V.dataColumn "category" (V.vStrs ["Ranger","Ranger","Ranger","Ranger","Ranger","Ranger","Ranger","Ranger"])


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
                            , V.maStroke [ V.vStr " rgb(80,127,98)"]
                            , V.maStrokeWidth [ V.vNum 2 ]
                            , V.maFill [ V.vStr " rgb(80,127,98)" ]
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

#### Sorcerer

```elm{v}
radarSorcerer : Spec
radarSorcerer =
    let
        table =
            V.dataFromColumns "table" []
                << V.dataColumn "key"(V.vStrs ["Conjuration", "Divination","Illusion", "Necromancy","Enchantment", "Transmutation", "Abjuration", "Evocation"])


                << V.dataColumn "value" (V.vNums [30,10,16,10,20,46,10,46])
                << V.dataColumn "category" (V.vStrs ["Sorcerer","Sorcerer","Sorcerer","Sorcerer","Sorcerer","Sorcerer","Sorcerer","Sorcerer"])


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
                            , V.maStroke [ V.vStr " rgb(153,46,46)"]
                            , V.maStrokeWidth [ V.vNum 2 ]
                            , V.maFill [ V.vStr " rgb(153,46,46)" ]
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

#### Warlock

```elm{v}
radarWarlock : Spec
radarWarlock =
    let
        table =
            V.dataFromColumns "table" []
                << V.dataColumn "key"(V.vStrs ["Conjuration", "Divination","Illusion", "Necromancy","Enchantment", "Transmutation", "Abjuration", "Evocation"])


                << V.dataColumn "value" (V.vNums [22,8,12,17,18,16,8,15])
                << V.dataColumn "category" (V.vStrs ["Warlock","Warlock","Warlock","Warlock","Warlock","Warlock","Warlock","Warlock"])


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
                            , V.maStroke [ V.vStr " rgb(123,70,155)"]
                            , V.maStrokeWidth [ V.vNum 2 ]
                            , V.maFill [ V.vStr " rgb(123,70,155)" ]
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

#### Wizard

```elm{v}
radarWizard : Spec
radarWizard =
    let
        table =
            V.dataFromColumns "table" []
                << V.dataColumn "key"(V.vStrs ["Conjuration","Divination","Illusion",  "Necromancy","Enchantment", "Transmutation", "Abjuration", "Evocation"])


                << V.dataColumn "value" (V.vNums [49,19,31,25,26,62,26,59])
                << V.dataColumn "category" (V.vStrs ["Wizard","Wizard","Wizard","Wizard","Wizard","Wizard","Wizard","Wizard"])


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
                            , V.maStroke [ V.vStr " rgb(42,80,161)"]
                            , V.maStrokeWidth [ V.vNum 2 ]
                            , V.maFill [ V.vStr " rgb(42,80,161)" ]
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
