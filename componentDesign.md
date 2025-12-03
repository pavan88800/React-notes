## ✅ 1. “Figure what would the consumer of this component want to know? What props shall we expose?”

## Meaning:

Your Otp component should think like a reusable component used by someone else.

If another developer imports <Otp />:

What options should they be able to control?

What values should they receive back?

Minimum props expected:

Examples:

```js
<Otp max={6} onChange={(value) => console.log(value)} />
```

So you should expose props like:

max → number of boxes

onChange → callback when OTP changes

Maybe: autoFocus, placeholder, isDisabled, etc.

## 👉 Reason: Write components that are flexible and reusable.

---

## ✅ 2. “Explore how popular libraries do OTP”

Meaning:

Look at libraries like:

react-otp-input

chakra-ui OTP

mui OTP

Learn:

What props they give

How they handle focus

How they validate input

How they style it

👉 Reason: Real-world comparison improves your component design.

---

## ✅ 3. “Why does Input component care about the whole OTP?”

Meaning:

Your Input component receives:

```js
otp;
setOtp;
inputRef;
index;
```

But it should not know global OTP state.

A good input component should only handle:

Its own value

Its own focus

Its own events

The parent (Otp) should manage the combined OTP and logic.

## 👉 Reason:

Follow the single-responsibility principle.
Input should be dumb. OTP parent should be smart.

---

## ✅ 4. “Think about Input in isolation — it shouldn't need to know its surroundings.”

Meaning:

Input should not:

Move focus to next input

Move focus to previous input

Modify entire array of otp

Those responsibilities belong to Otp.

Input should only:

Accept value

Call onChange

Better design:

```js
<Input
  value={otp[index]}
  isFocused={activeIndex === index}
  onChange={(value) => handleChange(index, value)}
/>
```

## 👉 Reason: Improves modularity and reusability.

---

## ✅ 5. “Logic for orchestrating input should be handled in OTP component”

Meaning:

Moving between input boxes:

Auto focus next

Handle backspace

Build final OTP string

This should happen in parent:

```js
function handleChange(index, value) {
  // update otp array
  // move focus
  // call onChange(completeOtpString)
}
```

## 👉 Reason: Keep child components simple and predictable.

---

## ✅ 6. “See if you can provide ‘isFocused’ prop in this input which handles auto focus”

Meaning:

Instead of using inputRef everywhere, you can do:

```js
<Input isFocused={index === activeIndex} />
```

Inside Input:

```js
useEffect(() => {
  if (isFocused) {
    ref.current.focus();
  }
}, [isFocused]);
```

## 👉 Reason: Cleaner and scalable autofocus logic.

---

## ✅ 7. “Check input attributes to ensure user types only numbers.”

Meaning:

You're blocking non-number values manually:

```js
if (isNaN(e.target.value)) return;
```

But HTML already has:

inputMode="numeric"

pattern="[0-9]\*"

maxLength={1} (but works only on type="text")

type="text" (not number, because number allows e, +, -)

Better input:

```js
<input type="text" inputMode="numeric" maxLength={1} />
```

## 👉 Reason: Do not reinvent what HTML gives for free.

---

## 🧠 TL;DR — What your senior is teaching you

Your current approach:

❌ All logic sits inside child component
❌ Components tightly coupled
❌ Hard to reuse
❌ Ref-heavy and complicated

What senior wants:

✔ Parent manages logic
✔ Child is dumb & reusable
✔ Clear props
✔ Clean architecture
✔ Input should not need full otp state
