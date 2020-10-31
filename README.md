# Pdx-Unlimiter Achievements

This repository contains the official achievements for the [Pdx-Unlimiter](https://github.com/crschnick/pdx_unlimiter)
and a reference for creating custom achievements.

## Creating Achievements

Pdx-Unlimiter achievements are specified in the [JSON format](https://en.wikipedia.org/wiki/JSON) and make heavy use
of [Json-Paths](https://github.com/json-path/JsonPath).
It is therefore useful to have a good understanding of both.

Custom achievements can be created in the `<pdxu_install_directory>/user_achievements/<game>/` directory.
To test your own Json-Path expressions, you can use the [Jayway JsonPath Evaluator](http://jsonpath.herokuapp.com/)
with the option `Always return result list` enabled.

The code that parses these achievement files can be found
[here](https://github.com/crschnick/pdx_unlimiter/blob/master/app/src/main/java/com/crschnick/pdx_unlimiter/app/achievement/)
If you believe that the Pdx-Unlimiter has an achievement related bug,
please report it on the [Pdx-Unlimiter repository](https://github.com/crschnick/pdx_unlimiter)
For anything else you can use the [issues page](https://github.com/crschnick/pdxu_achievements/issues) of this repository.

## Building

You can build the project with `gradle clean build`.
To create a release distribution, use `gradle createDist`.

## Pdxu Achievement Reference

An achievement file has to contain the following definition:

    {
      "achievement": {
        <properties>
      }
    }

The first two properties of an achievement are self-explanatory.
They specify information that every user can see like the name and description of the achievement:

    "name": "Lucky Lucca",
    "description": "As Lucca, own Lucknow!",

The next property is the uuid of the achievement:

    "uuid": "db341422-144a-11eb-adc1-0242ac120002",

This property is used to identify the achievement even if the name and description changes.
To generate this ID you can use an [online generator](https://uuid-generator.com/).

### Nodes

A node, in the context of pdxu achievements, are the contents of game data files or savegame files
on which Json-Path expressions can be applied on. (They don't have to be in the json format)

The savegame nodes are the contents of Pdx-Unlimiter save game data files, which are saved as `data.zip`.
To get an overview over the node structure of savegames, it is encouraged to browse the Pdx-Unlimiter save game data files.
You can also take a look at all [official achievements](https://github.com/crschnick/pdxu_achievements/tree/master/eu4/)
to get a better understanding of the pdxu achievement structure.

The game data nodes are the contents of selected game data files:
- `eu4.area`, the contents of `<EU4 install dir>/map/area.txt`
- `eu4.region`, the contents of `<EU4 install dir>/map/region.txt`
- `eu4.superregion`, the contents of `<EU4 install dir>/map/superregion.txt`
- `eu4.continent`, the contents of `<EU4 install dir>/map/continent.txt`
- `eu4.cultures`, the contents of `<EU4 install dir>/common/cultures/00_cultures.txt`

### Variables

Variables are defined with the following property:

    "variables": {
      <variableName>: <variable>
      ...
      <variableName>: <variable>
    },

There are three types of variables.
The first type are value variables, which store a fixed value that can be used later on:

      {
        "type": "value",
        "value": <value>,
      }
      
The second type are path value variables, which store the result of a Json-Path evaluation.
      
      {
        "type": "pathValue",
        "node": <node>,
        "path": <Json-Path>,
        "list": (true|false)
      }
       
If the list property is set to true, the variable will always store a list.
If the list property is set to false, the variable will try to unpack the list and return a value, which means that
if the variable evaluates to a list with the size of either zero or more than one, an exception is thrown.

The third type are path count variables, which store the size of a Json-Path evaluation:

      {
        "type": "pathCount",
        "node": <node>,
        "path": <Json-Path>,
      }

The return value of a variable evaluation can be referenced in other achievement definitions by using `%{variableName}` in the text.
There are also several predefined variables and the full list can be found
[here](https://github.com/crschnick/pdx_unlimiter/blob/master/app/src/main/java/com/crschnick/pdx_unlimiter/app/achievement/AchievementContent.java).
For example, the variable property could look like this:

    "variables": {
      "startDate": {
        "type": "pathValue",
        "node": "gamestate",
        "path": "$.start_date",
        "list": false
      },
      "provinceOwnedCount": {
        "type": "pathCount",
        "node": "provinces",
        "path": "$[?(@.owner == ${player})]"
      },
      "requiredProvinceCount": {
        "type": "value",
        "value": 100
      }
    },

### Icon

The icon property is optional and specifies which image should be displayed:

    "icon": "%{eu4.achievementImageDir}/achievement_lucky_lucca.dds",

To reference image files that lie in a specific directory you can use the following predefined variables:
- `%{eu4.achievementImageDir}`, which points to the achievement image directory of your EU4 installation
- `%{eu4.installDir}`, which points to the installation directory of the Pdx-Unlimiter.


### Conditions

Conditions make use of JsonPath filter expressions to evaluate to either true or false.
A condition node consists out of a description, an optional node entry and a JsonPath filter.

      {
        "description": <description>,
        "node": <node> (optional),
        "filter": <JsonPath filter>
      }

The node entry is only used if the JsonPath filter references it.
If a filter only uses variables that are already evaluated there is no need to specify a node entry:

      {
        "description": "Is Lucca",
        "filter": "[?(%{player} == 'LUC')]"
      }

Since the player variable was previously evaluated, the actual filter in use evaluates to `[?('HUN' == 'LUC')]` if the player is playing as Hungary.
However for more complex conditions, that cannot be completely replaced by variables, the node property is usually required:

      {
        "description": "Started as Lucca",
        "node": "countries_history",
        "filter": "[?($[%{player}]['tag_history'].length() == 0 || $[%{player}]['tag_history'][0] == 'LUC')]"
      }


There also exists several predefined conditions, which can be referenced with a text value.
The full list of predefined conditions can be found [here](https://github.com/crschnick/pdx_unlimiter/blob/master/app/src/main/java/com/crschnick/pdx_unlimiter/app/achievement/AchievementContent.java).



#### Achievement types

Achievement types define the different settings of an achievement, that make it impossible to compare the scores of two different players.
They consist out of a name and a list of conditions:

    "types": [
      {
        "name": <name>,
        "conditions": [
          <condition>
          ...
          <condition>
        ]
      },
      ...
      {
        "name": <name>,
        "conditions": [
          <condition>
          ...
          <condition>
        ]
      }
    ],


Common achievement types are the difficulty level as shown in the following example, which only supports the normal, hard and very hard difficulty:

    "types": [
      {
        "name": "Normal",
        "conditions": [
          "normal_difficulty"
        ]
      },
      {
        "name": "Hard",
        "conditions": [
          "hard_difficulty"
        ]
      },
      {
        "name": "Very hard",
        "conditions": [
          "very_hard_difficulty"
        ]
      }
    ]

If no set of conditions for any type is fulfilled, then the player is not eligible for the achievement.
For example if the player was playing on easy difficulty, no type conditions of the example above would be fulfilled.

### Eligibility conditions

The eligibility conditions specify whether a save game is currently eligible for the achievement.
The eligibility conditions are defined by a list of conditions:

    "eligibilityConditions": [
        <condition>,
        ...
        <condition>
    ]


Common eligibility conditions are usually the starting nation, the start date, the ironman setting and more.
For example the eligibility conditions could look like this:

    "eligibilityConditions": [
      "ironman",
      "normal_province_values",
      "historical_lucky_nations",
      "normal_start_date",
      "limited_country_forming",
      "no_custom_nation",
      {
        "description": "Started as Lucca",
        "node": "countries_history",
        "condition": "$[%{player}]['tag_history'].length() == 0 || $[%{player}]['tag_history'][0] == 'LUC'"
      }
    ]

Note that this example makes heavy use of the predefined conditions.

### Achievement conditions

Achievement conditions work exactly like eligibility conditions.
They specify the requirements to achieve the given achievement:

    "achievementConditions": [
        <condition>,
        ...
        <condition>
    ]

For example the achievement conditions could look like this:

    "achievementConditions": [
      "not_released_vassal",
      {
        "description": "Is Lucca",
        "condition": "%{player} == 'LUC'"
      },
      {
        "description": "Owns Lucknow",
        "condition": "%{lucknowOwner} == 'LUC'"
      },
      {
        "description": "Has core on Lucknow",
        "condition": "'LUC' in %{lucknowCores}"
      }
    ],

### Score

The score property defines how to calculate a score for the achievement:

    "score": <scorer>

A scorer can have a name and be one of five types.
The first type is a simple value scorer:

      {
        "name": <name> (optional),
        "value": <value>
      },

The second type is the condition scorer, which returns a value if all conditions are fulfilled:

      {
        "name": <name> (optional),
        "value": <value>,
        "conditions": [
          <condition>
          ...
          <condition>
        ]
      },

The third type is a path value scorer, which returns the value of a Json-Path evaluation.
This scorer throws an exception, if the result of the Json-Path evaluation can not be converted into a number.

      {
        "type": "pathValue"
        "name": <name> (optional),
        "path": <Json-Path>
      },

The fourth type is a path count scorer, which returns the size of the result of a Json-Path evaluation.

      {
        "type": "pathCount"
        "name": <name> (optional),
        "path": <Json-Path>
      },

The last type is a chain scorer, which is used to chain different scorers together.
The operator property specifies what operator will be used to chain the score values together.

      {
        "type": "chain"
        "operator": ("add" | "subtract" | "multiply" | "divide")
        "name": <name> (optional),
        "scorers": [
            <scorer>
            ...
            <scorer>
        ]
      },

