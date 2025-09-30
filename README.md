# CapoCompose
**Version:** 0.1 (draft)  
**Date:** 9/29/2025

CapoCompose is a text-based language for writing music notation that compiles to MNX. It lets you define instruments, create parts, arrange scores, and write music programmatically using intuitive syntax.
## A Complete Example
Here's a simple piano piece to show how everything fits together:
```CapoCompose
import key from scores

// Define a piano instrument
inst Piano {
    staff rh {
		clef "trebel"
	}
	staff lh {
		clef "bass"
	}
    name "Piano"
    shortName "Pno"
}

// Create a piano part from the instrument
part piano Piano {}

// Arrange the part in a score
score main {
    name "Piano Solo"
    parts [
	    piano
	]
}

// Call a function to set the key
key("C")

// Write music for the right hand
piano.rh {
    | 4c4 4d 4e 4f | 2g 2e | 1d |]
}

// Write music for the left hand
piano.lh {
    | 1c3 | 1g2 | 1g2 |]
}
```

This creates a complete MNX document with a three-measure piano piece.
## Instruments and Parts

**Instruments** are templates that define the structure of a musical part. They specify staff names, display names, and other properties:

```CapoCompose
inst Violin {
	staff {}
    name "Violin"
    shortName "Vln"
}
```

**Parts** are actual instances that appear in your score. Create a part from an instrument:

```CapoCompose
part vln1 Violin {}
```

Or define a part directly without creating an instrument first:

```CapoCompose
part vln2 {
	staff {}
    name "Violin 2"
    shortName "Vln2"
}
```

Parts can inherit from other parts, making it easy to create similar instruments:

```CapoCompose
part vln2 vln1 {
    name "Violin 2"      // Override just the name
    shortName "Vln2"
}
```
## Writing Music
Write music by opening a block with the part name (for single-staff parts) or staff reference (for multi-staff parts):

```CapoCompose
// Single staff
vln1 {
    | 4c5 4d 4e 4f | 2g 2e |
}

// Specific staff in multi-staff part
piano.rh {
    | 4c4 4d 4e 4f |
}
```

The pipe symbols `|` mark measure boundaries. The notation between pipes uses [Capo syntax](https://github.com/capo-lang/capo) - visit the Capo documentation to learn the note entry syntax.
## Scores
Scores define how parts are arranged on the page. At minimum, a score needs a name and a list of parts:

```CapoCompose
score fullScore {
    name "Full Score"
    parts [
        piano,
        vln1,
        vln2
    ]
}
```

Group related instruments with brackets using the `group` keyword:

```CapoCompose
score quartet {
    name "String Quartet"
    parts [
        group strings [
            vln1,
            vln2,
            vla,
            vc
        ]
    ]
}
```

When you create a score, CapoCompose automatically generates individual part scores for each instrument in addition to the full score you defined.

## Transposing Instruments

For transposing instruments, specify the transposition interval and whether to display written or concert pitch:

```CapoCompose
inst BbClarinet {
    staves [
        staff {}
    ]
    name "Clarinet in Bb"
    shortName "Cl."
    transpose -2        // Sounds 2 semitones lower than written
    useWritten true     // Display and write in transposed key
}
```

Set `useWritten` to `false` to write in concert pitch instead.

## Variables and Functions
Variables and functions let you reuse and transform musical content programmatically.
### Variables
Variables store reusable content - instruments, measures, note patterns, or Capo notation. Declare a variable with its type followed by a name:
```CapoCompose
// Instrument template
inst Violin {
    staves [staff {}]
    name "Violin"
}

// Measure structure
measure commonTimeMeasure {
    time {
        count 4
        unit 4
    }
}

// Capo notation pattern
capo theme {
    | 4c4 4d 4e 4f | 2g 2e | 2f 2d | 1c |
}
```

Use variables by referencing their name:
```CapoCompose
part vln1 Violin {}  // Create part from Violin template

vln1 {
    | {theme} |  // Insert the theme pattern
}
```
Variables containing Capo notation are evaluated in context - the same pattern can be used in different parts and will adapt to each context (transposition, clef, etc.).
### Functions
Functions transform or generate musical content. Define a function with parameters and return a value:

```CapoCompose
def transpose(capoCode, semitones) {
    for event in capoCode.content {
        if event.type == "note" {
            event.pitch.alter += semitones
        }
    }
    return capoCode
}

def repeat(capoCode, times) {
    result = capo {}
    for i in range(times) {
        result += capoCode
    }
    return result
}
```
Use functions to manipulate content before placing it in a part:
```
capo melody {
    | 4c4 4d 4e 4f |
}

vln1 {
    {melody}
    transpose(melody, 5)      // Up a perfect fourth
    repeat(melody, 3)          // Original melody three times
}
```
Functions can operate on any struct type - notes, measures, or complete capo blocks. They access struct properties using dot notation and can modify values before returning the transformed struct.
## What Happens Next

When you compile a CapoCompose file, the compiler:

1. Validates your syntax and structure
2. Converts Capo notation to MNX sequences and events
3. Generates measure timing and numbering
4. Creates the complete MNX document structure
5. Outputs valid MNX that can be rendered by any MNX-compatible application

The output includes all required MNX sections: `global` (timing, key signatures), `parts` (musical content), `layouts` (staff arrangements), and `scores` (page layouts).

## Next Steps

- **Learn Capo syntax:** Visit [github.com/capo-lang/capo](https://github.com/capo-lang/capo) for complete note entry documentation
- **MNX specification:** [w3c.github.io/mnx](https://w3c.github.io/mnx) to understand the output format
---
**CapoCompose is currently in early development and does not yet have a working implementation.** This documentation represents the design vision for the language, but the compiler, parser, and tooling do not exist yet. 

If you're interested in this project, I'd love your feedback and contributions:

- **Feedback on syntax and design:** Does this approach make sense? What's confusing? What's missing?
- **Use cases:** How would you want to use this? What features would make it valuable for your work?
- **Implementation help:** Interested in building parsers, compilers, or tooling? Reach out.

Feel free to start a discussion or open an issue. All input is welcome at this point.
