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

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.RecordTimeline;
using ActivityKitTests;

[Test(Pipeline)]
void a_quote_event_appears_on_its_opportunitys_activity() {
  var q = QuoteOf("Q-17");
  runas (Sam) { SendQuote(q); }
  Background.Settle();
  runas (Sam) { AcceptQuote(q); }
  Background.Settle();
  runas (Sam) {
    Assert.Equal(["Sam Ek took the yes on Q-17 fit-out phase one.", "Quote sent"], SaidOn(q.Id));
    var opp = Opportunity.Single(o => o.Name == "Harbour fit-out");
    Assert.Equal(["Sam Ek took the yes on Q-17 fit-out phase one. [Q-17 fit-out phase one]",
                  "Quote sent [Q-17 fit-out phase one]"], SaidOn(opp.Id));
  }
}
```

From [`tests/activity.test.osy`](tests/activity.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.RecordTimeline;
using ActivityKitTests;

[Test(Pipeline)]
void a_line_is_said_when_it_is_read() {
  var q = QuoteOf("Q-17");
  runas (Sam) { AcceptQuote(q); }
  Background.Settle();
  runas (Ola) {
    var renamed = Quote.Single(x => x.Id == q.Id);
    renamed.Title = "Q-17 fit-out, all phases";
    UnitOfWork.Commit();
  }
  runas (Sam) { Assert.Equal(["Sam Ek took the yes on Q-17 fit-out, all phases."], SaidOn(q.Id)); }
}
```

From [`tests/activity.test.osy`](tests/activity.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.RecordTimeline;
using ActivityKitTests;

[Test(Pipeline)]
void nobody_reads_the_activity_of_a_record_they_may_not_read() {
  var q = QuoteOf("Q-18");
  runas (Ola) { AcceptQuote(q); }
  Background.Settle();
  Guid opp = Guid.Empty;
  runas (Ola) {
    opp = Opportunity.Single(o => o.Name == "Board-only acquisition").Id;
    Assert.Equal(["Ola Strand took the yes on Q-18 due diligence."], SaidOn(q.Id));
    Assert.Equal(1, SaidOn(opp).Count);
  }
  runas (Sam) {
    Assert.Empty(SaidOn(q.Id));
    Assert.Empty(SaidOn(opp));
  }
}
```

From [`tests/activity.test.osy`](tests/activity.test.osy).

## Learn more

- [The Osysharp.RecordTimeline reference](https://osysharp.com/reference/ui/activity-kit/)
- [Osysharp.RecordTimeline in the package catalogue](https://osysharp.com/packages/#osysharp-recordtimeline)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
