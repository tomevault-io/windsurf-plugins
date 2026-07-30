---
trigger: always_on
description: Every conventional method — `Create()`, `Apply()`, and `ShouldDelete()` here, as well as the `Project()` / `Transform()` methods on an [EventProjection](/events/projections/event-projections) — takes the event it handles as one of its parameters. Marten determines *which* parameter is the event using the same rules for every projection type (`SingleStreamProjection`, `MultiStreamProjection`, and `EventProjection`):
---

# Aggregation with Conventional Methods

## How Marten Identifies the Event Argument

Every conventional method — `Create()`, `Apply()`, and `ShouldDelete()` here, as well as the `Project()` / `Transform()` methods on an [EventProjection](/events/projections/event-projections) — takes the event it handles as one of its parameters. Marten determines *which* parameter is the event using the same rules for every projection type (`SingleStreamProjection`, `MultiStreamProjection`, and `EventProjection`):

1. If a parameter is typed `IEvent<T>`, that parameter is the event and `T` is the event type. Use this when you want access to the [event metadata](/events/metadata).
2. Otherwise Marten looks for a single parameter whose type is a **concrete event type** — that is, not an interface such as `IQuerySession` / `IDocumentOperations`, not `IEvent`, not `CancellationToken`, and not the aggregate type. If exactly one parameter qualifies, it is the event. **This is the usual case, and the parameter can be named anything.**
3. If a method has more than one parameter that could be the event (an ambiguous signature that type inference can't resolve), Marten falls back to the parameter **name**: a parameter named `@event`, `event`, `e`, or `ev` is taken as the event.

In practice you almost never need to think about this. `Apply(SomeEvent e, MyAggregate aggregate)` just works, because `SomeEvent` is the only concrete, non-aggregate parameter — the name `e` is incidental, not required. The name convention only matters to disambiguate the unusual signature that has more than one candidate parameter.

::: tip
The recognized event parameter names are `@event`, `event`, `e`, and `ev`. You only ever need one of these when type inference alone cannot pick out the event parameter.
:::

## Aggregate Creation

::: tip
As of Marten 7, if your aggregation projection has both a `Create()` function or constructor for an event type, and
an `Apply()` method for the same event type, Marten will only call one or the other method depending on whether the
aggregate already exists **but never both** for one single event.
:::

Aggregates can initially be created behind the scenes by Marten if there's a no-arg constructor function on the aggregate
document type -- which doesn't have to be public by the way.

You can also use a constructor that takes an event type as shown in this sample of a `Trip` stream aggregation:

<!-- snippet: sample_trip_stream_aggregation -->
<a id='snippet-sample_trip_stream_aggregation'></a>
```cs
public class Trip
{
    // Probably safest to have an empty, default
    // constructor unless you can guarantee that
    // a certain event type will always be first in
    // the event stream
    public Trip()
    {
    }

    // Create a new aggregate based on the initial
    // event type
    internal Trip(TripStarted started)
    {
        StartedOn = started.Day;
        Active = true;
    }

    public Guid Id { get; set; }
    public int EndedOn { get; set; }

    public double Traveled { get; set; }

    public string State { get; set; }

    public bool Active { get; set; }

    public int StartedOn { get; set; }
    public Guid? RepairShopId { get; set; }

    // The Apply() methods would mutate the aggregate state
    internal void Apply(Arrival e) => State = e.State;
    internal void Apply(Travel e) => Traveled += e.TotalDistance();

    internal void Apply(TripEnded e)
    {
        Active = false;
        EndedOn = e.Day;
    }

    // We think stream aggregation is mostly useful for live aggregations,
    // but hey, if you want to use a aggregation as an asynchronous projection,
    // you can also specify when the aggregate document should be deleted
    internal bool ShouldDelete(TripAborted e) => true;
    internal bool ShouldDelete(Breakdown e) => e.IsCritical;
    internal bool ShouldDelete(VacationOver e) => Traveled > 1000;
}
```
<sup><a href='https://github.com/JasperFx/marten/blob/master/src/DaemonTests/TestingSupport/TripProjectionWithCustomName.cs#L118-L168' title='Snippet source file'>snippet source</a> | <a href='#snippet-sample_trip_stream_aggregation' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

Or finally, you can use a method named `Create()` on a projection type as shown in this sample:

<!-- snippet: sample_tripprojection_aggregate -->
<a id='snippet-sample_tripprojection_aggregate'></a>
```cs
public partial class TripProjection: SingleStreamProjection<Trip, Guid>
{
    // These methods can be either public, internal, or private but there's
    // a small performance gain to making them public
    public void Apply(Arrival e, Trip trip) => trip.State = e.State;

    public void Apply(Travel e, Trip trip)
    {
        Debug.WriteLine($"Trip {trip.Id} Traveled " + e.TotalDistance());
        trip.Traveled += e.TotalDistance();
        Debug.WriteLine("New total distance is " + e.TotalDistance());
    }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JasperFx/marten](https://github.com/JasperFx/marten) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
