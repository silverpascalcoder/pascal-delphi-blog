+++
date = '2026-08-12T17:52:00+10:00'
draft = false
title = 'Five Sins of Exception Handling in Object Pascal'
summary = 'A companion piece to Tim’s “Five Sins in Five Minutes” video on Exceptions located on Youtube'
+++

# Five (Six) Sins of Exception Handling in Object Pascal

 This is a companion piece to Tim’s “Five Sins in Five Minutes” video on Exceptions and found on Youtube (https://youtu.be/0qxkhr3EPGM?si=Vr9vRtbQ9nfF-EqC). All examples below are in Free Pascal / Lazarus (Object Pascal) or Delphi, but applicable to any language that uses exception handling.

Exceptions exist for *exceptional* situations. They are not part of normal control flow. Once upon a time they were also expensive to set up and tear down; even today the real cost is usually lost context and harder debugging. The patterns below are ones that appear repeatedly in real code reviews and public repositories.

## 1. Swallowing Exceptions

```pascal
procedure Example1;
begin
  try
    SomeCriticalOperation;
  except
    // empty
  end;
end;
```

The error is completely hidden. You lose the original exception type, message, stack, and any chance of diagnosing what went wrong. If you catch an exception you must *do* something useful with it (log with context, clean up, translate, or re-raise).

## 2. Reraising Without Adding Value

```pascal
procedure Example2;
begin
  try
    SomeCriticalOperation;
  except
    raise;   // or just "raise;"
  end;
end;
```

This is pure ceremony. The `try…except` block adds nothing: no logging, no cleanup, no extra context. Delete the whole construct and let the exception propagate naturally from the point it was raised.

## 3. Catching Exceptions Too Broadly

```pascal
procedure Example3;
begin
  try
    SomeCriticalOperation;
  except
    on E: Exception do
      // handle everything the same way
  end;
end;
```

Catching the root `Exception` (or worse, a bare `except`) treats every failure as equal. A network timeout, an out-of-memory condition, a logic invariant break, and a simple input mistake all land in the same handler. Prefer the most specific exception type you can handle, and let the rest propagate.

## 4. Using Exceptions When a Non-Throwing Alternative Exists

```pascal
procedure Example4;
var
  N: Integer;
begin
  try
    N := StrToInt(SomeString);   // throws on bad input
  except
    N := 0;
  end;
end;
```

`StrToInt` is the classic example. Free Pascal already provides safer alternatives:

```pascal
N := StrToIntDef(SomeString, 0);

// or
if not TryStrToInt(SomeString, N) then
  N := 0;
```

The same principle applies to file handling, parsing, etc.: prefer the “Try…” or “…Def” forms, or perform an explicit check *before* the operation that would raise. Exceptions should not be used for ordinary validation.

## 5. Missing Context When Logging

```pascal
procedure Example5;
begin
  try
    SomeCriticalOperation;
  except
    on E: Exception do
      Log(E.Message);           // or worse…
      // Log('Oops! We have a problem.');
  end;
end;
```

Knowing that “file not found” occurred is only half the story. *Where* in the program did it happen? What parameters were involved? A bare message (or a generic “Oops”) forces the next developer to guess. Add enough context that the log entry is useful on its own:

```pascal
on E: Exception do
  Log(Format('SomeCriticalOperation failed for user %s: %s',
             [UserName, E.Message]));
```

## Bonus (the sixth sin)

Capturing the exception object and then deliberately ignoring its contents (logging a fixed string instead) is essentially a more polite form of swallowing. You paid the cost of the exception machinery and still threw the diagnostic information away.

---

**Guiding principle**

Exceptions are for exceptional situations. If you can detect the problem with ordinary control flow or with a non-throwing helper (`Try…`, `…Def`, explicit existence checks, etc.), do that instead. When you *do* catch an exception, add value—logging with context, resource cleanup, or translation into a more appropriate exception—otherwise let it travel upward unmolested.

These patterns are language-agnostic in spirit, but the concrete examples above are written in the Object Pascal dialect used by Free Pascal and Lazarus.
