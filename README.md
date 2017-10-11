# Unreal 4 Coding Conventions 1.0

This document summarizes the high-level coding conventions for writing Unreal 4 client code at Daedalic Entertainment. They are based on the official Unreal Coding Standard:

* https://docs.unrealengine.com/latest/INT/Programming/Development/CodingStandard/index.html

The goal is to make it easier to work in similar teams inside and outside the company, as well as have client code blend in with other code of the Unreal API. We are providing a complete summary here in order to allow people to understand the conventions at a glance, instead of having to open multiple documents. Our coding conventions are numbered, which makes it easier to refer to them in code reviews.

In case we've missed recent changes to the official Unreal Coding Standard, or you can spot any other issue, please [create a pull request](https://help.github.com/articles/creating-a-pull-request/).


## 1. Namespaces

1.1. __DO NOT__ use namespaces to organize your game classes. Namespaces are not supported by UnrealHeaderTool, so they can't be used when defining `UCLASS`es, `USTRUCT`s etc., and we don't want to put half of our game code in a namespace and the other half in global scope.

1.2. __DO__ add a prefix to all class and struct names, based on your project name.


## 2. Files

2.1. __DO__ use PascalCase for file names, with the project name prefix, but without the type prefix (e.g. `AHOATCharacter` goes into `HOATCharacter.h`).

2.2. __DO__ write header files with the following structure:

* `#pragma once`
* `#include` of the pre-compiled header, if any (e.g. `#include "HOATPCH.h"`)
* `#include` of the base class header, if any (e.g. `#include "GameFramework/Character.h"`)
* `#include` of the generated class header (e.g. `#include "HOATCharacter.generated.h"`)
* delegate declarations
* forward declarations for any referenced engine or game types
* type definition

2.3. __DO__ define classes with the following structure:

* constructors
* destructor
* public `override` functions (e.g. `BeginPlay`, `Tick`, `GetLifetimeReplicatedProps`)
* public functions
* public event handlers
  * `virtual void Notify...` functions
  * `UFUNCTION(BlueprintImplementableEvent) void Receive...` functions
  * `UPROPERTY(BlueprintAssignable)` delegate properties
* operators
* public constants
* public `UPROPERTY`s
* protected `override` functions
* protected functions
* protected constants
* protected fields
* private functions
* private constants
* private fields

Within each of these groups, order members by name or logical groups.

2.4. __DO NOT__ cover more than a single feature in each file. Don't define more than one public type per file.

2.5. __DO__ leave a blank line at the end of the file to play nice with gcc. 


## 3. Includes

3.1. __DO NOT__ include unused headers. This will generally help reduce the compilation time, especially for developers when just one header has been modified. It may also avoid errors that can be caused by conflicts between headers. If an object in the class is only used by pointer or by reference, it is not required to include the header for that object. Instead, just add a forward declaration before the class. 

3.2. __DO NOT__ rely on a header that is included indirectly by another header you include.


## 4. Classes & Structs

4.1. __DO__ use PascalCase for class names, e.g. `AHOATCharacter`.

4.2. __DO__ add type prefixes to class names to distinguish them from variable names. For example, `FSkin` is a type name, and `Skin` is an instance of a `FSkin`. UnrealHeaderTool requires the correct prefixes in most cases, so it's important to provide them.

* Template classes are prefixed by `T`. 
* Classes that inherit from `UObject` are prefixed by `U`. 
* Classes that inherit from `AActor` are prefixed by `A`. 
* Classes that inherit from `SWidget `are prefixed by `S`. 
* Classes that are abstract interfaces are prefixed by `I`. 
* Enums are prefixed by `E`. 
* Most other classes are prefixed by `F`, though some subsystems use other letters. 

4.3. __DO__ prefix boolean variables by `b` (e.g. `bPendingDestruction` or `bHasFadedIn`).

4.4. __DO__ uppercase the first letter of acronyms, only, e.g. `FHOATXmlStreamReader`, not `FHOATXMLStreamReader`.

4.5. __DO__ add a virtual destructor to classes with virtual member functions.

4.6. __DO__ mark classes that are not meant to be derived from as `final`. This should be the default for non-interface classes. Care has to be taken when removing the `final` keyword from a class when inheritance is required. Classes that are already derived don't need to be marked as `final` by default: In the most common case there is no reason to prevent further inheritance.

4.7. __DO__ use a non-virtual destructor in `final` classes unless they are already derived.

4.8. __DO__ use `struct`s for data containers, only. They shouldn't contain any business logic beyond simple validation or need any destructors.


## 5. Constructors

5.1. __DO__ mark each constructor that takes exactly one argument as `explicit`, unless it's a copy constructor or the whole point of the constructor is to allow implicit casting. This minimizes wrong use of the constructor.


## 6. Functions

6.1. __DO__ use PascalCase for functions names.

6.2. __DO NOT__ add a space between function name and parameter list:

      // Right:
      void Tick(float DeltaSeconds);

      // Wrong:
      void Tick (float DeltaSeconds);

6.3. __DO__ pass each object parameter that is not a basic type (`int`, `float`, `bool`, `enum`, or pointers) by reference-to-const. This is faster, because it is not required to do a copy of the object. Also, this is required for exposing the property in blueprints:

      bool GetObjectsAtWorldPosition(const FVector& WorldPositionXY, TArray<FHitResult>& OutHitResults);

6.4. __DO__ prefix function parameter names with `Out` if they are passed by reference and the function is expected to write to that value. This makes it obvious that the value passed in this argument will be replaced by the function. If an `Out` parameter is also a boolean, put the `b` before the `Out` prefix, e.g. `bOutResult`. 

6.5. __DO__  flag methods as `const` if they do not modify the object.

6.6. __CONSIDER__ writing functions with six parameters or less. For passing more arguments, try and use `structs` instead, and/or refactor your function.

6.7. __CONSIDER__ using enum values instead of boolean function parameters.

      // Right:
      ShowMessageBox(TEXT("Nice Title"), TEXT("Nice Text"), MessageBox::MESSAGEBOX_BUTTONS_OK);

      // Wrong: Meaning of third parameter is not immediately obvious.
      ShowMessageBox(TEXT("Nice Title"), TEXT("Nice Text"), false);

6.8. __AVOID__ providing function implementations in header files. Use inline functions judiciously, as they force rebuilds even in files which don't use them. Inlining should only be used for trivial accessors and when profiling shows there is a benefit to doing so. Be even more conservative in the use of FORCEINLINE. All code and local variables will be expanded out into the calling function and will cause the same build time problems caused by large functions. Don't use inlining or templates for functions which are likely to change over a hot reload.


## 7. Variables

7.1. __DO__ use PascalCase for variable names.

7.2. __AVOID__ short or meaningless names (e.g. `A`, `Rbarr`, `Nughdeget`). Single character variable names are only okay for counters and temporaries, where the purpose of the variable is obvious.

7.3. __DO NOT__ use negative names for boolean variables.

    // Right:
    if (bVisible)

    // Wrong: Double negation is hard to read.
    if (!bInvisible)

7.4. __DO__ declare each variable on a separate line so that a comment on the meaning of the variable can be provided.  Also, the JavaDoc style requires it.

7.5. __DO__ use portable aliases for basic C++ types:

* `bool` for boolean values (_never_ assume the size of `bool`). `BOOL` will not compile. 
* `TCHAR` for a character (_never_ assume the size of `TCHAR`). 
* `uint8` for unsigned bytes (1 byte). 
* `int8` for signed bytes (1 byte). 
* `uint16` for unsigned "shorts" (2 bytes). 
* `int1` for signed "shorts" (2 bytes). 
* `uint32` for unsigned ints (4 bytes). 
* `int32` for signed ints (4 bytes). 
* `uint64` for unsigned "quad words" (8 bytes). 
* `int64` for signed "quad words" (8 bytes). 
* `float` for single precision floating point (4 bytes). 
* `double` for double precision floating point (8 bytes). 
* `PTRINT` for an integer that may hold a pointer (_never_ assume the size of `PTRINT`). 

7.6. __DO__ put a single space between the `*` or `&` and the variable name for pointers or references, and don't put a space between the type and `*` or `&`. For us, the fact that we are declaring a pointer or reference variable here much more belongs to the type of the variable than to its name:

      AController* Instigator
      const FDamageEvent& DamageEvent

7.7. __DO__ test whether a pointer is valid before dereferencing it. `nullptr` should be used instead of the C-style `NULL` macro in all cases. If the pointer points to any `UOBJECT`, use `IsValid` to ensures the pointer is not null and the referenced object is not marked for destruction.


## 8. Enums & Constants

8.1. __CONSIDER__ using `enum class` over `static constexpr` over `static const` variables over `#define` when defining constants.

8.2. __DO__ use ALL_CAPS with underscores between words for constant names.


## 9. Indentation & Whitespaces

9.1. __DO__ use four spaces for indentation.

9.2. __DO__ use a single space after a keyword and before a parenthesis.

      // Right:
      if (bVisible)
      {
      }

      // Wrong:
      if(bVisible)
      {
      }

9.3. __DO__ surround binary operators with spaces.

9.4. __DO NOT__ put multiple statements on one line.


## 10. Line Breaks

10.1. __CONSIDER__ keeping lines shorter than 100 characters; wrap if necessary.

10.2. __DO__ use a new line for the body of a control flow statement:

      // Right:
      if (bVisible)
      {
          Hide();
      }

      // Wrong:
      if (bVisible) Hide();

10.3. __DO__ start operators at the beginning of the new lines.

      // Right:
      if (longExpression
          + otherLongExpression
          + otherOtherLongExpression)
      {
      }

      // Wrong: Operator at the end of the line is easy to miss if the editor is too narrow.
      if (longExpression +
          otherLongExpression +
          otherOtherLongExpression)
      {
      }


## 11. Braces

11.1. __DO__ put opening braces on their own lines:

      // Right:
      if (bVisible)
      {
      }
      else
      {
      }

      // Wrong:
      if (bVisible) {
      } else {
      }

11.2. __DO__ have the left brace on the start of a line for class declarations and function definitions:

      void AHOATCharacter::Tick(float DeltaSeconds)
      {
      }

      class Moo
      {
      };

11.3. __DO__ use curly braces, even if the body of a conditional statement contains just one line:

      // Right:
      if (bVisible)
      {
          Hide();
      }

      // Wrong: Can lead to subtle bugs in the future, if the body is extended to span multiple statements.
      if (bVisible)
          Hide();


## 12. Parentheses

12.1. __DO__ use parentheses to group expressions:

      // Right:
      if ((a && b) || c)

      // Wrong: Operator precedence is not immediately clear.
      if (a && b || c)

12.3. __DO NOT__ use spaces after parentheses:

      // Right:
      if ((a && b) || c)

      // Wrong:
      if ( ( a && b ) || c )


## 13. Control Flow

13.1. __DO__ add a `break` (or `return`) statement at the end of every `case`, or a comment to indicate that there's intentionally no `break`, unless another `case` follows immediately within switch statements

      switch (MyEnumValue)
      {
        case Value1:
            DoSomething();
            break;

        case Value2:
        case Value3:
            DoSomethingElse();
            // Fall through.

        default:
            DefaultHandling();
            break;
      }

13.2. __DO NOT__ put `else` after jump statements:

      // Right:
      if (ThisOrThat)
      {
          return;
      }

      SomethingElse();


      // Wrong: Causes unnecessary indentation of the whole else block.
      if (ThisOrThat)
      {
          return;
      }
      else
      {
          SomethingElse();
      }


13.3. __DO NOT__ mix const and non-const iterators.

      // Right:
      for (Container::const_iterator it = c.cbegin(); it != c.cend(); ++it)

      // Wrong: Crashes on some compilers.
      for (Container::const_iterator it = c.begin(); it != c.end(); ++it)


## 14. Language Features

14.1. __CONSIDER__ using the `auto` keyword when it avoids repetition of a type in the same statement, or when assigning iterator types. If in doubt, for example if using `auto` could make the code less readable, do not use `auto`.

      auto* HealthComponent = FindComponentByClass<UHOATHealthComponent>();

14.2. __DO__ use `auto*` for auto pointers, to be consistent with references, and to add additional guidance for the reader.

14.3. __DO__ use proprietary types, such as `TArray` or `TMap` where possible. This avoids unnecessary and repeated type conversion while interacting with the Unreal Engine APIs.

14.4. __DO__ use the `TEXT()` macro around string literals. Without it, code which constructs `FString`s from literals will cause an undesirable string conversion process. 

## 15. Events & Delegates

15.1. __CONSIDER__ exposing meaningful events to subclasses and/or other interested listeners by defining virtual functions and/or multicast delegates.

15.2. __DO__ define two functions when exposing an event to a subclass. The first function should be virtual and its name should begin with `Notify`. The second function should be a `BlueprintImplementableEvent UFUNCTION` and its name should begin with `Receive`. The default implementation of the virtual function should be to call the `BlueprintImplementableEvent` function (see `AActor::NotifyActorBeginOverlap` and `AActor::ReceiveActorBeginOverlap` for example).

15.3. __DO__ call the virtual function before broadcasting the event, if both are defined (see `UPrimitiveComponent::BeginComponentOverlap` for example).

Example:

    DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(FHoatActorGraphConnectivityChangedSignature, AActor*, Source, AActor*, Target, float, Distance);

    /** Event when the connectivity of an observed source vertex has changed. */
    virtual void NotifyOnConnectivityChanged(AActor* Source, AActor* Target, float Distance);

    /** Event when the connectivity of an observed source vertex has changed. */
    UFUNCTION(BlueprintImplementableEvent, Category = Graph, meta = (DisplayName = "OnConnectivityChanged"))
    void ReceiveOnConnectivityChanged(AActor* Source, AActor* Target, float Distance);

    /** Event when the connectivity of an observed source vertex has changed. */
    UPROPERTY(BlueprintAssignable)
    FHoatActorGraphConnectivityChangedSignature OnConnectivityChanged;


    void AHoatActorGraph::NotifyOnConnectivityChanged(AActor* Source, AActor* Target, float Distance)
    {
    ReceiveOnConnectivityChanged(Source, Target, Distance);
    OnConnectivityChanged.Broadcast(Source, Target, Distance);

    HOAT_LOG(hoat, Log, TEXT("%s changed the connectivity of vertex %s: Distance to target %s changed to %f."),
            *GetName(), *Source->GetName(), *Target->GetName(), Distance);
    }

## 16. Comments

16.1. __DO__ add a space after `//`.

16.2. __DO__ place the comment on a separate line, not at the end of a line of code.

16.3. __DO__ write API documentation with [Javadoc comments](https://docs.unrealengine.com/latest/INT/Programming/Development/CodingStandard/index.html#exampleformatting).


## 17. Additional Naming Conventions

17.1. __DO NOT__ use any swearing in symbol names, comments or log output.
