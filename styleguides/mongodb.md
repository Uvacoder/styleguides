# MongoDB Style Guide

## Table of Contents

  1. [General](#general)
  1. [Enumerations](#enumerations)
  1. [Booleans](#booleans)
  1. [Dates](#dates)
  1. [Null and undefined](#null-and-undefined)
  1. [Other types](#other-types)
  1. [Names](#names)
  1. [Object modelling](#object-modelling)


## General

<a name='general--no-surprises'></a>
- [1.1](#general--no-surprises) **Avoid surprises**: Your goal when creating a new schema should be to [be as least surprising as you can](https://en.m.wikipedia.org/wiki/Principle_of_least_astonishment). The rules in this style guide are written with this in mind. If there is a trade off between ease of creating data and ease of understanding its structure, choose the latter.

    > Why? Having others be able to understand your data is usually more important than making your code slightly more convenient.


<a name='general--priorities'></a>
- [1.2](#general--priorities) **Priorities**: There are several dimensions that will impact your database design, such as used hard disk space, read/write speed and ease of development. Spend some time thinking about which ones are important in your case. Then come to the conclusion that ease of development is the only thing that matters because you're not Google.

    > Why? It's easy to fall into the trap of wasting time making things scale from the start. This makes it less likely that you will ever get to a point where these problems become real.

## Enumerations

An enumeration is a type that allows a limited number of values, e.g. a `role` field that can contain the values `'DOCTOR'`, `'NURSE'`, `'PATIENT'` and `'ADMINISTRATOR'`.

<a name='enumerations--modelling'></a>
- [2.1](#enumerations--modelling) **Modelling**: Always model your enums as uppercase string constants, e.g. `'WAITING'`, `'IN_PROGRESS'` and `'COMPLETED'`.

    > Why?
    > 1. Favoring strings over booleans or numbers means your data is self-documenting. If your `gender` field contains the value `1`, you don't know if that means male, female or something else.
    > 1. Using uppercase strings makes it easy to see at a glance whether something is an enum or a user-facing value.

    | :x: gender        | :x: gender        | :white_check_mark: gender |
    | :---------------- | :---------------- | :----------------- |
    | 0                 | 'Male'            | 'MALE'             |
    | 1                 | 'Female'          | 'FEMALE'           |

<a name='enumerations--explicit'></a>
- [2.2](#enumerations--explicit) **Explicit states**: Don't use `null`, `undefined`, or any value except upper case string constants in your enums. This includes initial, undecided or unknown states.

    > Why? It makes meaning explicit. If you have an assessmentState field that's set to `null`, you don't know whether that means 'no assessment necessary', 'not applicable', 'patient didn't show up', 'undecided' or any other possible state.

    | :x: assessmentState | :white_check_mark: assessmentState |
    | :------------------ | :--------------------------------- |
    | null                | 'NOT_REQUIRED'                     |
    | *missing*           | 'NOT_APPLICABLE'                   |


## Booleans

<a name='booleans--booleans-and-enums'></a>
- [3.1](#booleans--booleans-and-enums) **Booleans and enums**: Don't model things as booleans that have no natural correspondence to true/false, even if they only have two possible states. Use enums instead.

    > Why?
    > 1. Booleans can not be extended beyond two possible values, e.g. a boolean field 'hasArrived' could not be changed later to include the possibility of cancelled appointments.

    | :x: gender              | :white_check_mark: gender |
    | :---------------- | :----------------- |
    | false             | 'MALE'             |
    | true              | 'FEMALE'           |

<a name='booleans--prefix'></a>
- [3.2](#booleans--prefix) **Prefix**: Prefix your boolean field names with verbs such as 'is...' or 'has...' (e.g. 'isDoctor', 'didAnswerPhoneCall' or 'hasDiabetes').

<a name='booleans--orthogonal'></a>
- [3.3](#booleans--orthogonal) **Orthogonality**: If you have several mutually exclusive boolean fields in your collection, merge them into an enum.

    > Why? So that it's impossible to save invalid data in your db, like a car that's green and red simultaneously.

    :x:

    | isRed             | isBlue             | isGreen            |
    | :---------------- | :----------------- | :----------------- |
    | false             | true               |  false             |
    | true              | false              | false              |

    :white_check_mark:

    | color             |
    | :---------------- |
    | 'BLUE'            |
    | 'RED'             |

## Dates

<a name='dates--iso-strings'></a>
- [4.1](#dates--iso-strings) **ISO strings**: Make sure you never save dates as ISO strings like `'2017-04-14T06:41:21.616Z'`. This can easily happen as a result of JSON deserialization.

    | :x:              | :white_check_mark: |
    | :---------------- | :----------------- |
    | '2017-04-14T06:41:21.616Z'             | ISODate('2017-04-14T06:41:21.616Z')            |

    
<a name='dates--day-strings'></a>
- [4.2](#dates--day-strings) **Day strings**: Don't use the Date type when all you are concerned with is the day component. Instead, use strings of the form 'YYYY-MM-DD'

    > Why? It makes common operations much easier, like checking whether a date falls on a certain day or getting all the documents that fall on a certain day.

    | :x: dateOfBirth             | :white_check_mark: dateOfBirth |
    | :---------------- | :----------------- |
    | ISODate('1989-10-03T06:41:21.616Z')             | '1989-10-03'            |

## Null and undefined

<a name='null-and-undefined--no-overloading'></a>
- [5.1](#null-and-undefined--no-overloading) **No overloading**: Don't overload the meaning of `null` and `undefined` to mean anything other than 'unset'.

    > Why? It breaks expectations and makes it impossible to interpret the data by looking at it.

<a name='null-and-undefined--sensible-default'></a>
- [5.2](#null-and-undefined--sensible-default) **Default**: Don't use null or undefined when there is a sensible default value like `0`, `''` or `[]`.

    > Why? It keeps your types pure

    | :x: notes             | :white_check_mark: notes |
    | :---------------- | :----------------- |
    | 'Some interesting observation'             | 'Some interesting observation'           |
    | null             | ''           |

    | :x: comments             | :white_check_mark: comments |
    | :---------------- | :----------------- |
    | [ 'First!', 'Great post!' ]             | [ 'First!', 'Great post!' ]    |
    | null             | []           |

<a name='null-and-undefined--primitive-types'></a>
- [5.3](#null-and-undefined--primitive-types) **Primitive types**: In columns that contain primitive types (booleans, numbers or strings) or Date objects, use `null` to express absent values.

    | :x:            | :white_check_mark: |
    | :---------------- | :----------------- |
    | 1             | 1   |
    | 2             | 2          |
    | *missing*             | null           |

<a name='null-and-undefined--objects-and-arrays'></a>
- [5.4](#null-and-undefined--objects-and-arrays) **Complex types**: To express the absence of a value in columns that contain objects or arrays, add another column that determines whether your array or object is present. If it isn't, it should be undefined.

    :x:

    | productType              | chapterTitles |
    | :---------------- | :----------------- |
    | 'BOOK'             | ['1 Get Started', '2 The End'] |
    | 'CAR'             | []         |
    | 'CAR'             | null         |

    :white_check_mark:

    | productType              | chapterTitles |
    | :---------------- | :----------------- |
    | 'BOOK'             | ['1 Get Started', '2 The End'] |
    | 'CAR'             | *missing*        |
    | 'CAR'             | *missing*         |

<a name='null-and-undefined--dont-mix'></a>
- [5.5](#null-and-undefined--dont-mix) **Don't mix the two**: Don't mix `null` and `undefined` in the same column.

    > Why? It makes it hard to understand what the two values are supposed to represent.

    | :x: height            | :white_check_mark: height |
    | :---------------- | :----------------- |
    | 178             | 178   |
    | null             | null          |
    | *missing*             | null           |

<a name='null-and-undefined--multiple-missing-value-states'></a>
- [5.6](#null-and-undefined--multiple-missing-value-states) **Multiple missing value states**: If you need two ways to represent the absence of a value, convert the value to an object with `state` and `value` keys.

    :x:

    | weight                          |
    | :------------------------------ |
    | null                            |
    | *missing*                       |
    | 97                              |
    | ''                              |

    :white_check_mark:

    | weight                          |
    | :------------------------------ |
    | { state: 'NOT_APPLICABLE' }     |
    | { state: 'USER_DOES_NOT_TELL' } |
    | { state: 'SET', value: 97 }     |

## Other types

<a name='other-types--mix'></a>
- [6.1](#other-types--mix) **Mixed types**: Don't mix values of different types in one column. Restrict yourself to one type per column.

    > Why? So you don't have to typecheck or cast when you consume the data.

    | :x:           |:white_check_mark: |
    | :------------ | :---------------- |
    | 1             | 1                 |
    | '2'           | 2                 |
    | { value: 3 }  | 3                 |

<a name='other-types--object-schema'></a>
- [6.2](#other-types--object-schema) **Object columns**: If you have a column or array that contains objects, make sure all objects share the same schema.

    | :x:       | :white_check_mark:     |
    | :---------------- | :------------- |
    | { value: 42 }    | { value: 42 }   |
    | { otherValue: 'foo' } | { value: 43 }   |
    | { }              | { value: null } |

<a name='other-types--falsiness'></a>
- [6.3](#other-types--falsiness) **Falsiness**: Don't use `''` or `0` for their falsiness.

    > Why? It makes it very easy to write buggy code.

    | :x: height      | :white_check_mark: height  |
    | :---------------- | :------------- |
    | 182            | 182         |
    | 167            | 167         |
    | 0               | null          |

<a name='other-types--numbers-as-strings'></a>
- [6.4](#other-types--numbers-as-strings) **Numbers as strings**: Don't save numbers as strings except when saving data like phone numbers or ID numbers for which leading zeroes are significant and for which arithmetic operations don't make sense.

    > Why? It means you don't have to cast when performing arithmetic or when comparing values.

    | :x: height             | :white_check_mark: height |
    | :---------------- | :----------------- |
    | '178'             | 178           |
    | '182'             | 182           |

    | :x: phoneNumber             | :white_check_mark: phoneNumber |
    | :---------------- | :----------------- |
    | 20123123            |  '020123123'   |

<a name='other-types--numbers-with-unit'></a>
- [6.5](#other-types--numbers-with-unit) **Units**: If you really need to include units with your numbers (think about whether you do, can't you convert and save them in metric?), save them as objects with a `unit` and a `value` field.

    > Why? It makes it possible to perform comparisons and arithmetic operations on your values.

    | :x:              | :white_check_mark:  |
    | :--------------- | :----------------- |
    | '10 kg'          | { unit: 'KG', value: 10 }          |
    | '20 stones'      | { unit: 'STONES', value: 20 }          |

<a name='other-types--sets'></a>
- [6.6](#other-types--sets) **Sets**: Model sets as arrays containing uppercase string constants. Use JavaScript's `Set` class in your code where appropriate.

    > Why? You can look at sets as multi-valued enumerations.

    | :x:              | :white_check_mark:  |
    | :---------------- | :----------------- |
    | ['Dolphin', 'Pigeon', 'Bee']             | ['DOLPHIN', 'PIGEON', 'BEE']          |
    | { DOLPHIN: true, PIGEON: false, BEE: false }             | ['DOLPHIN']           |

<a name='other-types--object-ids'></a>
- [6.7](#other-types--object-ids) **ObjectIds**: Don't use MongoDB's ObjectId type. Instead, set _id fields yourself, either by using a property of your data that is naturally unique or by creating random strings.

    > Why? Serializing and deserializing ObjectIds is just too much of a hassle and easily leads to bugs.

    | :x: _id             | :white_check_mark: _id |
    | :---------------- | :----------------- |
    | ObjectId("5937a6a76cb02c00018577fe")             | '6xAySKn98aZ66vN'          |
    | ObjectId("5937a7136cb02c00018577ff")             | 'eOiga4lkLaW99ER'          |

## Names

<a name='names--abbreviations'></a>
- [7.1](#names--abbreviations) **Abbreviations**: Don't use abbreviations except for domain specific language. When you do abbreviate, capitalize properly.

    > Why? It makes your data and code hard to read.

    - :x: `apTime`
    - :white_check_mark: `appointmentTime`
    - :x: `ankleBrachialPressureIndexRight`
    - :white_check_mark: `ABIRight`
    - :x: `healthInformationSystemNumber`
    - :x: `hisNumber`
    - :white_check_mark: `HISNumber`


<a name='names--key-names'></a>
- [7.2](#names--key-names) **Case**: Use camelCase over snake_case for key names.

    > Why? It's what we use in JavaScript which means we don't have to convert or mix the two in our code.

<a name='names--collection-names'></a>
- [7.3](#names--collection-names) **Collection names**: Collection names should be pluralized and in camelCase. Use dots when there is a relationship between collections (e.g. `users` and `users.appointments`)

## Object modelling

<a name='object-modelling--growth'></a>
- [8.1](#object-modelling--growth) **Growth**: Don't let your objects keep growing. Prune and merge properties into nested objects where appropriate (e.g. by combining 'footAssessmentState', 'eyeAssessmentState' and 'nutritionAssessmentState' into a nested 'assessmentStates' object with properties 'foot', 'eye' and 'nutrition')

<a name='object-modelling--excessive-nesting'></a>
- [8.2](#object-modelling--excessive-nesting) **Nesting**: Don't excessively nest objects. Consider breaking up your data if you find yourself needing deeply nested objects

## Todo (pull requests welcome)

- Normalization/denormalization
