# Osysharp.RecordTimeline

What happened to a record, on its own timeline. Every event your app and its kits raise about a record becomes a line
on it, said in the reader's words. A record that others roll up to shows what happened to them too, so an
opportunity's timeline carries its quotes'. Each line is shown only to somebody who may read the record.

**Use case.** A sales team works opportunities and the quotes made for them. A quote is sent, then accepted. Both are
lines on the quote and on its opportunity, with the rep's name and the quote's current title. A rep who may not see a
board-only deal sees nothing of its timeline. The same kit puts a ticket's history on a ticket, or an order's on an
order: it never knows what the record is.

```osy
use Osysharp.RecordTimeline@0;

policy ReadsActivity => IsRep;                    // who reads a record's activity

[Event("Quote accepted")]
public sealed class QuoteAccepted {               // any event, from your app or any kit
  public Quote Quote;
  public Person? By;
  public override string Sentence() { return $"{By.DisplayName} took the yes on {Quote.Title}."; }
}

entity Quote {
  [RollsUpTo] Opportunity Opportunity;            // its events are told on the opportunity too
  …
}

ActivityTimeline(quote.Id);                       // on the record's page
```

## What it does

| | |
|---|---|
| **Every event, no wiring** | the kit hears every event raised in the app — your own internal ones too; each is a line on the record it is about (its first record member). Because a timeline wants them all, an app that uses it pays one delivery for each piece of work that raised an event |
| **Rolled up** | a reference marked `[RollsUpTo]` puts the record's lines on the record it points at as well, naming where they came from |
| **Said when read** | a line keeps the event and says it with the event's own `Sentence()`, in the reader's language, with its records read as the reader may read them: a renamed record reads with its new name |
| **Only for who may read** | the timeline of a record the reader may not read is empty; an event whose records they may not see says its label alone |
| **Kept for a declared time** | a year by default; `app.Activity = new ActivitySetup { Keep = TimeSpan.FromDays(90) }` shortens it, and an hourly sweep keeps to it. A line goes with its record |
| **One component** | `ActivityTimeline(record.Id)`, newest first, on whatever page shows the record |
| **Personal data** | what an event carried is kept classified as content |

## Its tests

Every behaviour above is proven against a small sales pipeline: a quote's events on its opportunity, a line said as it
is read, a private deal nobody else sees the timeline of, the retention sweep, and the timeline on the record's page.
