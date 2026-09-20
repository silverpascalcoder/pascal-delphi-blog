+++
date = '2026-09-20T10:32:00+10:00'
draft = false
title = 'Five in Five #1: Common Pascal Mistakes'
summary = 'Welcome to the first proper episode of the Five in Five series. The goal is simple: cover five common mistakes that both new and experienced programmers make when working with Free Pascal, Lazarus, or Delphi.''
+++

# Common Free Pascal, Lazarus & Delphi Mistakes That Hurt Your Applications

Welcome to the first proper episode of the *Five in Five* series. The goal is simple: cover five common mistakes that both new and experienced programmers make when working with Free Pascal, Lazarus, or Delphi — and do it quickly.

These are practical issues I have seen. Fixing them improves startup time, maintainability, and the overall quality of your applications.

---

## 1. Auto-Creating Every Form

When you create a new form in Lazarus or Delphi, it is automatically added to the project’s auto-create list. You end up with code like this in your `.lpr` file:

```pascal
Application.Initialize;
Application.CreateForm(TForm1, Form1);
Application.CreateForm(TForm2, Form2);
Application.CreateForm(TForm3, Form3);
Application.Run;
```

The main form needs to be created. Secondary forms (About boxes, Options dialogs, rarely used windows) usually do not.

**Why this is a problem:**
- Every auto-created form consumes memory at startup.
- Any code in `FormCreate` (loading settings, opening queries, etc.) runs even if the user never opens that form.
- Startup time increases unnecessarily.

**Better approach:**
- Turn off “Auto-create new forms” in Project Options → Forms.
- Or simply remove the unwanted `Application.CreateForm` lines.
- Create forms only when needed:

```pascal
procedure TForm1.ShowOptionsClick(Sender: TObject);
var
  OptionsForm: TOptionsForm;
begin
  OptionsForm := TOptionsForm.Create(Self);
  try
    OptionsForm.ShowModal;
  finally
    OptionsForm.Free;
  end;
end;
```

---

## 2. Leaving Database Components Active at Design Time

It is very convenient during development to set a `TSQLQuery.Active := True` or a connection’s `Connected := True` so you can see data in the form designer or in a `TDBGrid`.

**The danger:**  
When you move the application to another machine, a test environment, or a customer’s computer, the database path or connection settings are often different. The application can fail to start or throw errors that make you look unprofessional.

**Better approach:**
- Leave `Active` and `Connected` set to `False` at design time.
- Load connection settings from a configuration file (or environment-specific settings) at runtime and open the connection/query only when needed.

This keeps your project portable and avoids hard-coded paths that only work on your development machine.

---

## 3. Using Default Component Names

It is easy to drop controls on a form and leave the names as `Button1`, `Button2`, `Edit1`, `Edit2`, etc.

Later in the code you see:

```pascal
procedure TForm1.Button1Click(Sender: TObject);
begin
  // What does this actually do?
end;
```

You (or the next developer) have to switch back to the form just to remember which button is which.

**Better approach:**  
Give controls meaningful names as soon as you create them:

- `btnOK`
- `btnCancel`
- `btnHelp`
- `edtPhoneNumber`
- `edtEmail`

Then the event handler becomes self-documenting:

```pascal
procedure TForm1.btnOKClick(Sender: TObject);
begin
  // Immediately clear what this does
end;
```

This small habit pays off enormously as the form grows.

---

## 4. Putting All Logic Inside Event Handlers

Because it is so easy to double-click a button and start typing, many developers end up with large amounts of business logic directly inside `OnClick` handlers.

This creates several problems:
- The UI code and business logic become tightly mixed.
- The same logic cannot be easily reused.
- The event handler becomes long and hard to test.

**Better approach:**  
Treat event handlers as thin coordinators. Keep them focused on UI concerns and move real work into separate methods or, better still, into dedicated units.

```pascal
procedure TForm1.btnOKClick(Sender: TObject);
begin
  if ValidateInput then
    SaveCustomer;
end;
```

The actual saving logic lives elsewhere and can be called from multiple places.

---

## 5. Copy-Pasting the Same Logic

Imagine you have code that adds a phone number to the database. You write it in one form’s `btnOKClick`. Later you need the same logic in another form (contacts, vendors, bulk import…). The fastest solution feels like copying and pasting the code.

Months later you need to add a new field (e.g. phone type: mobile/work/home). Now you have to find and update every pasted copy.

**Better approach:**  
As soon as you notice duplication, extract the logic into its own unit:

```pascal
unit PhoneService;

interface

procedure AddPhoneNumber(const ANumber, AType: string);

implementation

procedure AddPhoneNumber(const ANumber, AType: string);
begin
  // Single place that talks to the database
end;

end.
```

All forms simply call `AddPhoneNumber`. One place to maintain, one place to test, one place to improve.

---

### Summary

| # | Mistake                        | Better Practice                              |
|---|--------------------------------|----------------------------------------------|
| 1 | Auto-creating every form       | Create forms only when needed                |
| 2 | Active DB components at design time | Open connections/queries at runtime     |
| 3 | Default component names        | Use meaningful names (`btnOK`, `edtPhone`)  |
| 4 | Logic inside event handlers    | Separate UI from business logic              |
| 5 | Copy-paste code                | Extract shared logic into its own unit       |

These five habits cost almost nothing to adopt and save a lot of time (and embarrassment) later.

This is the first episode of the *Five in Five* series. Next week we’ll look at another five common issues.

If you have mistakes you frequently see (or ones you used to make yourself), leave them in the comments — they may appear in a future episode.

Happy coding.
