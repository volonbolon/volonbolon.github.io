---
layout: post
title: "Importing symbols from ObjC to Swift"
date: 2018-03-09 23:32:56 GMT
tags: swift objective-c tips
---

Following the excellent [First-Class Swift API for Objective-C Frameworks](https://pspdfkit.com/blog/2018/first-class-swift-api-for-objective-c-frameworks/), I’ve rediscovered the pretty useful `NS_TYPED_ENUM` and `NS_TYPED_EXTENSIBLE_ENUM`. They are macros used in Objective-C in order to expose to Swift a bunch of constants as if they belongs to the same type. 

Constants that represent a fixed set of possible values can be imported as a structure by adding the `NS_TYPED_ENUM` macro. 

```
// Constants.h
typedef NSString *BachSonata NS_TYPED_ENUM;

FOUNDATION_EXPORT BachSonata const BWV1001; // Sonata No. 1 in G minor
FOUNDATION_EXPORT BachSonata const BWV1002; // Partita No. 1 in B minor
FOUNDATION_EXPORT BachSonata const BWV1003; // Sonata No. 2 in A minor
FOUNDATION_EXPORT BachSonata const BWV1004; // Partita No. 2 in D minor
FOUNDATION_EXPORT BachSonata const BWV1005; // Sonata No. 3 in C major
FOUNDATION_EXPORT BachSonata const BWV1006; // Partita No. 3 in E major

// Constants.m
BachSonata const BWV1001 = @"Sonata No. 1 in G minor";
BachSonata const BWV1002 = @"Partita No. 1 in B minor";
BachSonata const BWV1003 = @"Sonata No. 2 in A minor";
BachSonata const BWV1004 = @"Partita No. 2 in D minor";
BachSonata const BWV1005 = @"Sonata No. 3 in C major";
BachSonata const BWV1006 = @"Partita No. 3 in E major";
```

And, once in swift, we can switch the values as if there were any other `enum`: 

```
func play(sonata: BachSonata) {
    switch sonata {
    case .BWV1001:
        print("Sonata No. 1 in G minor")
    case .BWV1002:
        print("Partita No. 1 in B minor")
    case .BWV1003:
        print("Sonata No. 2 in A minor")
    case .BWV1004:
        print("Partita No. 2 in D minor")
    case .BWV1005:
        print("Sonata No. 3 in C major")
    case .BWV1006:
        print("Partita No. 3 in E major")
    default:
        print("This is not a sonata")
    }
}

play(sonata: .BWV1004)
```

Which, as you can see, is pretty convenient. But, the best part is yet to come. If you decorate the `typedef` as `NS_TYPED_EXTENSIBLE_ENUM`, then, you can extend it later. 

```
// Constants.h
typedef NSString *REMRecords NS_TYPED_EXTENSIBLE_ENUM; 
// Hey, hope is the last thing you lose.

FOUNDATION_EXPORT REMRecords const Murmur;
FOUNDATION_EXPORT REMRecords const Reckoning;
FOUNDATION_EXPORT REMRecords const FablesOfTheReconstruction;
FOUNDATION_EXPORT REMRecords const LifesRichPageant;
FOUNDATION_EXPORT REMRecords const Document;

// Constants.m
REMRecords const Murmur = @"Murmur";
REMRecords const Reckoning = @"Reckoning";
REMRecords const FablesOfTheReconstruction = @"Fables of the Reconstruction";
REMRecords const LifesRichPageant = @"Lifes Rich Pageant";
REMRecords const Document = @"Document";

// Swift.swift
extension REMRecords {
    static var Green: REMRecords {
        return REMRecords(rawValue: "Green")
    }
}

func play(record: REMRecords) {
		print(record.rawValue)
}

play(record: .Green)
```