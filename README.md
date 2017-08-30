# The Mask: Core library for the Masking technology

The Mask provides a contextual dependency lookup engine on steroids.

## <a name="synopsis"></a>Synopsis

Masking helps building complex (large scale) dynamic applications. It offers a
much simpler alternative for most creational and behavioral design patterns.
This makes building complex software fun and saves a lot of time.

## <a name="when_to_use"></a>When to use

When building applications that have to deal with:
- customer customization
- lots of variation in data and process
- frequent change

## <a name="how_it_works"></a>How it works

It's all about match making. Just like finding the perfect date. Only this time
we're looking for dependencies.

Dependencies can be any type of object: data, business logic, views, etc..
Each dependency can have multiple candidates (a.k.a. variants). Each candidate
has it's own unique set of selection criteria.

Picking the best candidate for a dependency depends on the application's
context. The candidate that corresponds most with the context wins.

Masks provide matching information for the context and the candidates.
The context uses masks as circumstances. Candidates use masks as selection
criteria.

## <a name="installation"></a>Installation

This library requires Java 7 or higher.
To add the dependency using Maven, add the following to the POM:

```xml
<dependency>
  <groupId>org.maskingtech</groupId>
  <artifactId>TheMask</artifactId>
  <version>0.1.0</version>
</dependency>
```

## <a name="basic_usage"></a>Basic usage

All we need is Mike, the masking assistant. He provides everything needed to
fully utilize the technology. Who he is, nobody knows. But that’s not important.
Just let him handle it and all will be OK.

### <a name="obtaining_mike"></a>Obtaining a Mike object

The easiest way to obtain a Mike object is to use a builder. This library
provides a basic builder to get you up and running in no time.

```java
Builder builder = new BasicBuilder();
Mike mike = builder.build("example");
```

### <a name="creating_masks"></a>Creating masks

Masks are named and hold a map of criteria used as matching information.
Criteria have an [operation](#operations), value and a
[value type](#value_types). Once created, masks must be registered into the mask
register provided by Mike.

The example below shows how to create and register masks.

```java
// Create mask
Mask happyHourMask = new Mask("happyHour");
happyHourMask.addCriterion("time.hour", Operation.EQUALS, "5", Double.class);
happyHourMask.addCriterion("time.period", Operation.EQUALS, "pm", String.class);

// Register mask
MaskRegister masks = mike.getMasks();
masks.registerMask(happyHourMask);
```

### <a name="activating_masks"></a>Activating masks

Registered masks can be activated and deactivated using Mike.

Moeten we nog wel vertellen dat wanneer je meerdere maskers zet en je weghaalt, dat ie in 
feite alleen de dependencies die behoren tot dat masker weghalen? Het is niet zo dat ie 
bijhoudt welke maskers er actief zijn en welke circumstances daarbij horen.

Peter: Dan moeten we meer uitleggen over het achterliggende systeem (als in, maskers zijn
niet meer dan simpele lijsen van circumstances, en zo worden zo ook behandeld).

```java
// Activate
mike.setMask("happyHour");

// Deactivate
mike.removeMask("happyHour");
```

### <a name="creating_dependencies"></a>Creating dependencies

There is only one rule: each dependency MUST implement the Dependency interface.
It doesn't matter if the interface has been inherited.

The example below shows how to create dependencies using inheritance.

```java
public interface Discount extends Dependency
{
    public double calculate(double price);
}

public class NoDiscount implements Discount
{
    @Override
    public double calculate(double price)
    {
        return 0; // 0% off, sorry
    }
}

public class HappyHourDiscount implements Discount
{
    @Override
    public double calculate(double price)
    {
        return price * 0.2; // 20% off
    }
}

```

### <a name="register_dependencies"></a>Register dependencies

Dependencies are registered in the dependency register. They can be registered
in the root of the register, or in a [subset](#working_with_sets).

Once a dependency has been registered candidates can be added that can be picked
based on the application's context.

The example below shows how to register dependencies and candidates in a set.

```java
DependencyRegister dependencies = mike.getDependencies();
DependencySet examplesSet = dependencies.registerSet("examples");

CandidateList candidates = examplesSet.registerDependency("Discount", "All type of discounts");
candidates.registerCandidate("org.maskingtech.examples.NoDiscount");
candidates.registerCandidate("org.maskingtech.examples.HappyHourDiscount", happyHourMask);
```

### <a name="requesting_dependencies"></a>Requesting dependencies

Once all dependencies and their candidates are registered, they can be
[requested in several ways](#creating_sharing_dependencies).

The example below shows how to request shared dependencies without and with
a mask.

```java
double price = 10.0;

Discount sample1 = mike.share("examples", "Discount");
System.out.println(String.format("Normal discount %f", sample1.calculate(price)));

mike.setMask("happyHour");

Discount sample2 = mike.share("examples", "Discount");
System.out.println(String.format("Happy hour discount %f", sample2.calculate(price)));
```

This results in the following.

```
Normal discount 0.000000
Happy hour discount 2.000000
```

Based on the output we can conclude that requesting the Discount dependency
without the happyHour mask Mike returned the NoDiscount candidate. After setting
the mask Mike returned the HappyHourDiscount candidate.

### <a name="complete_example"></a>Complete example

Here's the complete example application based on the snippets above.

```java
package org.maskingtech.examples;

import org.maskingtech.core.Mike;
import org.maskingtech.core.building.BasicBuilder;
import org.maskingtech.core.building.Builder;
import org.maskingtech.core.common.Mask;
import org.maskingtech.core.common.Operation;
import org.maskingtech.core.register.CandidateList;
import org.maskingtech.core.register.DependencyRegister;
import org.maskingtech.core.register.DependencySet;
import org.maskingtech.core.register.MaskRegister;

public class Example
{
    public static void main(String[] args)
    {
        // Step 1: obtain a Mike object
        
        Builder builder = new BasicBuilder();
        Mike mike = builder.build("example");
        
        // Step 2: create masks
        
        Mask happyHourMask = new Mask("happyHour");
        happyHourMask.addCriterion("time.hour", Operation.EQUALS, "5", Double.class);
        happyHourMask.addCriterion("time.period", Operation.EQUALS, "pm", String.class);

        MaskRegister masks = mike.getMasks();
        masks.registerMask(happyHourMask);
        
        // Step 3: register dependencies
        
        DependencyRegister dependencies = mike.getDependencies();
        DependencySet examplesSet = dependencies.registerSet("examples");
        
        CandidateList candidates = examplesSet.registerDependency("Discount", "All type of discounts");
        candidates.registerCandidate("org.maskingtech.examples.NoDiscount");
        candidates.registerCandidate("org.maskingtech.examples.HappyHourDiscount", happyHourMask);
        
        // Step 4: request dependencies
        
        double price = 10.0;
        
        Discount sample1 = mike.share("examples", "Discount");
        System.out.println(String.format("Normal discount %f", sample1.calculate(price)));
        
        mike.setMask("happyHour"); // Activate the mask
        
        Discount sample2 = mike.share("examples", "Discount");
        System.out.println(String.format("Happy hour discount %f", sample2.calculate(price)));
    }
}
```

## Advanced usage

### <a name="creating_sharing_dependencies"></a>Creating vs sharing dependencies

There are two options when it comes to requesting dependencies.

The first option is by using Mike's share(setName, dependencyName) method as
we've seen in the examples above. This method creates a single instance for each
unique candidate. If the same dependency within the same context is requested
multiple times, Mike will return the single instance of the candidate over and
over again. This will come in handy when you want to save memory or need to
share a single instance with other application components, like service objects.

The second option is by using Mike's create(setName, dependencyName) method.
This method creates a new instance for every request. Use this method when you
want to make sure that every instance is unique, like data objects.

### <a name="working_with_sets"></a>Working with sets

Sets are named collections of unique dependencies. They can be used for grouping
dependencies, to avoid dependency naming conflicts, or both.

Each set can have it's own collection of subsets to provide more levels of
grouping.

Registering sets should be done using the dependency register provided by Mike.
The example below show how to register (sub)sets.

```java
// Get the dependency register
DependencyRegister dependencies = mike.getDependencies();

// Register simple set
DependencySet utilitiesSet = dependencies.registerSet("utilities");

// Register nested sets
DependencySet productDataSet = dependencies.registerSet("product.models");
DependencySet productViewSet = dependencies.registerSet("product.views");
DependencySet productDataSet = dependencies.registerSet("product.controllers");
```

Note that the dot (.) character is used for creating subsets. There is no need
for registering the product set before registering the subsets.

Using sets is not mandatory but we recommend to use them for applications that
have more then a few dependencies.

### <a name="manipulating_context"></a>Manipulating the context

So far we only introduced the usage of masks for defining matching information.
When applying to the context, masks can have some limitations due to their
static nature. In some cases this is not sufficient.

For example:
The example application above shows how to deal with discounts. The selection
of the appropriate discount candidate depends on the current hour. This means
that the context must contain the current time to make this work.

For these type of situations the context must be manipulated by setting
circumstances manually. The example below shows how Mike helps to accomplish
this.

```java
// Get the current hour
String hour = "3"; // imagine some real implementation here

// Set the hour as circumstance
mike.setCircumstance("hour", hour);
```

Note that the value of the hour is defined as a string. Mike will figure out
the appropriate value type used for matching the value. An extra parameter can
be added to the setCircumstance(...) method to specify the value type manually.
We do not recommend this. As said before, let Mike handle it.

### <a name="working_with_scopes"></a>Working with scopes

During the execution of the application circumstances can change permanently and
temporarily. In case of temporarily changes scopes can be used to ease tracking
and rolling back all context modifications.

For example:
Imagine that the example application must now work with different discounts per
type of product. Calculating the total price for all products requires picking
the correct discount candidate for each product. To accomplish this the current
type of product must be set as a circumstance in the context. To prevent
unexpected behavior after calculating the the total the type of product must be
removed from the context.

The following example shows how to use scopes to handle this situation.

```java
double total = 0.0;

mike.openScope();

for (Product product : products)
{
    mike.setCircumstance("productType", product.getType());

    Discount discount = mike.share("examples", "Discount");

    double price = product.getPrice();
    double discount = discount.calculate(price);

    total += price - discount;
}

mike.closeScope();

System.out.println(String.format("Total %f", total));
```

Keep in mind that opening a scope is simply a copy of the list of circumstances.
This means that circumstances can also be removed from the scope freely without
losing them forever. The only way to restore the list of circumstances is to
close the newly created scope. The changes in the new scope do not affect the
old scope.

### <a name="versioning_dependencies"></a>Versioning dependencies

Many applications are exposed to new insights and business changes frequently.
This forces these applications to endure continuous change. This effects the
implementation of the data that flows through the application and the business
process logic.

When existing processes change many challenges present themselves if new
processes require additional information. In the best case scenario a migration
strategy can be set up to keep the data compatible with the newest release. In
the worst case scenario a migration strategy is not an option. This requires
complex mechanisms to support both the old and new processes simultaneously.

In the worst case scenario versioning dependencies helps addressing this
problem. The versioning system enables applications to run multiple versions
simultaneously. When utilizing the context properly it is possible to mix old
parts with new parts gracefully to slowly phase out all old versions.

```java
CandidateList candidates = examplesSet.registerDependency("Discount", "All type of discounts");

candidates.registerCandidate("org.maskingtech.examples.HappyHourDiscount", happyHourMask);
candidates.registerCandidate("org.maskingtech.examples.HappyHourDiscountV2", happyHourMask, new Version(2, 0, 0));
```

Versions use the following numbering system: major, minor, patch

## <a name="options"></a>Options

So far, the information in this document only focused on the straight forward, simple 
options that the library has to offer.
In the next chapter the full array of options will be discussed. These are more advanced
options for a broader array of matching options, tracing the runtime execution and
handling exceptions. All of these options help you to better utilize the mask for building
richer customer applications.

### <a name="operations"></a>Operations

The matching engine uses operations to match candidates to the given circumstances.

- EQUALS
- NOT_EQUALS
- GREATER
- GREATER_OR_EQUALS
- LESS
- LESS_OR_EQUALS
- BETWEEN
- OUTSIDE

The equals has been shown in the examples above, and is straight forward. A candidate
with a criterion of equals will only be selected when the circumstance has the exact same
value as the candidate criterion.

The not equals works the opposite of the equals, and matches if the criterion value is not
exactly equal to the circumstance value.

The operations greater, greater or equals, less and less or equals are all straight
forward, and do what the operation name defines.

Between and outside exclude the boundary values when matching. For example, a condition
where the weight of the fruit is between 5 and 12 ounces, fruit weighing 5 or 12 ounce
are not considered a candidate.

### <a name="value_types"></a>Value types

The matching engine uses three types of values for matching.

- Date
- Double
- String

Only criteria of the same value type will be checked. This means that a condition stating
the color as green (string) and a candidate with color as 0080000 (Double) won't match. 
The value types are different and therefore the candidate is completely ignored.

The operations mentioned above are executed against these value types. The String value
type is the only type that only supports the equals and not equals operations. 

### <a name="exceptions"></a>Exceptions

The mask throws several type of exceptions. The main type of exception is the 
MaskingException and is actually never thrown.
Further more, the ContextException, LoaderException, MatchingException and 
RegisterException are all subclasses, which are also not thrown. These exceptions help to
determine the area of error.

MaskingException
- InvalidVersionException
- InvalidMaskNameException

ContextException
- CloseContextRootScopeException

LoaderException
- DependencyClassNotFoundException
- DependencyInstantiationException
- UnexpectedDependencyTypeException

MatchingException
- MultipleCandidatesFoundException
- NoCandidateFoundException
- NoEvaluatorFoundException
- UnsupportedMatchOperationException

RegisterException
- DependencyNotFoundException
- MaskAlreadyDefinedException
- MaskNotFoundException
- MultipleCandidatesFoundException
- SetNotFoundException

HashingException
- HashAlgorithmNotFoundException

Of course, the developer can decide at which level the application should catch what
types of exceptions. For example, when building a RESTful api, the 
NoCandidateFoundException could be used for returning a 'bad request', but the
MatchingException could also be used for it.

```java
Discount discount;

try
{
    discount = mike.share("examples", "Discount");
    
    //some real implementation
}
catch(NoCandidateFoundException)
{
	throw new NotFoundException(exception.getMessage());
}

```

### <a name="events"></a>Events

The engine is a complex library of code, which doesn't immediately shows the execution
path of your application. This is the cost of being able to dynamically execute objects.
To help you in your search for actual execution, each important event throws an event to
an event manager. Event listeners can be registered to listen to these events and take
the appropriate action.

The events defined are;
- CandidateFoundEvent
- CandidateRegisteredEvent
- CircumstanceRemovedEvent
- CircumstanceSetEvent
- CircumstancesClearedEvent
- ContextScopeClosedEvent
- ContextScopeOpenedEvent
- ContextSubScopesClosedEvent
- DependencyRegisteredEvent
- MaskRegisteredEvent
- MaskRemovedEvent
- MaskSetEvent
- NoCandidateFoundEvent
- SetRegisteredEvent
- HashCalculcatedEvent

### <a name="event_listener"></a>Event manager

The event manager handles the events that are thrown by the library. It uses an
event bus to publish events on and requires event listeners to listen to these
events. As stated in the section above, the library already throws events to the
event manager, but the event listener interface is not implemented. For listening
to events, the event listener needs to be implemented manually.

### <a name="design_principles"></a>Design principles

- Remove unnecessary circumstances as soon as possible (use scopes!)
- Matching exact values for value type Double (Numeric) should be used with caution,
  because of the nature of the double precision.

## <a name="known_limitations"></a>Known limitations

(Missing priorities for criteria, thus easy to create multiple candidate exceptions)
(StringEvaluator needs to be added last to the evaluator register)

## <a name="options"></a>Terminology

- Candidate - a variant of a dependency bound to contextual selection criteria.
- Circumstance - a condition that must be met for a candidate to be considered for 
  selection.
- Criterion - a rule for evaluating matches.
- Context - a set of circumstances used for matching possible candidates.
- Dependency - something the application depends on like a model, view or controller.
- Mask - provides matching information (criteria) for the context and the candidates.
- Mike - central point for accessing the masking functionality.
- Operation - a predefined way for matching criteria.
- Set - named collections of unique dependencies.
- Scope - a set of circumstances in the context. Scopes are unaffected by each other.
- Value type - the type of a circumstance values, i.e. String, Date, Number.
- Version - order mechanism to sort equally qualified candidates.
