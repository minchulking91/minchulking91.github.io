---
title: "EditText에 천단위 콤마 붙이기 Best Practice"
date: 2019-06-06 10:00:00 +0900
categories: android edittext textwatcher 화폐단위 천단위 editable
---

목적 : 에딧텍스트에 숫자가 바뀔때마다 3자리 숫자 단위로 Grouping separator(,) 를 찍어준다.

```kotlin
package kr.co.sleeptime_grt.edittextprac

import android.text.Editable
import android.text.TextWatcher

class DecimalFormatTextWatcher : TextWatcher {

    companion object {
        private val decimalRegex = Regex("""([1-9]\d*|0)(\.\d*)?""")
        private val digitsRegex = Regex("""[^.\d]""")
        private const val DECIMAL_SEPARATOR = '.'
    }

    private var busy: Boolean = false

    override fun afterTextChanged(s: Editable?) {
        if (busy || s.isNullOrEmpty()) return
        busy = true
        val decimalString = s.replace(digitsRegex, "") //사용하는 digits 필터링.
            .let { decimalRegex.find(it)?.value } //decimal string 찾음.
            ?.let { decimalString ->
                //정수부에 , 추가.
                if (decimalString.contains(DECIMAL_SEPARATOR)) {
                    //정수부와 소수부 나눔.
                    val (intPart, fractionPart) = decimalString.split(DECIMAL_SEPARATOR)
                    formattedIntPart(intPart).append(DECIMAL_SEPARATOR).append(fractionPart)
                } else {
                    //정수부만 있음.
                    formattedIntPart(decimalString)
                }.toString()
            }
            ?: ""
        s.replace(0, s.length, decimalString)
        busy = false
    }

    override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {
        //no-op
    }

    override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
        //no-op
    }

    /**
     * 123456789 -(reverse)-> 987654321 -(add comma every 3 digits)-> 987,654,321 -(reverse)-> 123,456,789
     */
    private fun formattedIntPart(intPart: CharSequence): StringBuilder {
        val temp = StringBuilder()
        intPart.reversed().forEachIndexed { index, c ->
            if (index > 0 && index.rem(3) == 0) {
                temp.append(",")
            }
            temp.append(c)
        }
        return temp.reverse()
    }
}
```

afterTextChanged() 에서 editable을 수정하면 TextWatcher 에 EditText 를 추가하지 않아도 되므로 좀더 안전하게 쓸 수 있다.

  숫자를 입력하기 위해서 보통 EditText 에 inputType 을 number 혹은 number|numberDecimal 을 쓸텐데 이때 문제가 좀 있다. ```,``` 를 입력 할 수 없기 때문이다.

```java
//DigitsKeyListener
...
//min sdk 26 이하에서 사용하는 number digits 
private static final char[][] COMPATIBILITY_CHARACTERS = {
    { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' },
    { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '-', '+' },
    { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '.' },
    { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '-', '+', '.' },
};
...
@Deprecated
public DigitsKeyListener()
...
//min sdk 26부터 사용이 가능하다.
public DigitsKeyListener(@Nullable Locale locale) 
```

  inputType으로 number|numberDecimal 를 쓸 경우 DigitsKeyListener 를 keyListener 로 넣게 되는데 ```,```의 입력을 허용해주지 않기 때문이다. 또 낮은 버전의 안드로이드 경우 number을 쓸 때 Locale에 따라 소수점 반영이 안돼 ```,```를 소수점으로 입력할 수 없다. 만약 Locale 을 신경쓰지 않는다면 digits String 을 넣고 KeyListener 를 만들면 된다.

```kotlin
editText.keyListener = DigitsKeyListener.getInstance("0123456789,.")
```

  Locale 을 신경 써야 한다면 좀 더 복잡해지는데 DigitsKeyListener 를 sdk 레벨 제한을 낮춰서 Backport 를 만들어야 하기 때문이다. DigitsKeyListener 에는 android 패키지의 Locale이 들어가는데 sdk 24 부터 지원하므로 그 이하라면 코드를 복사해서 java Locale 을 사용하도록 수정하면 된다. 그리고 위에 만들어둔 TextWatcher 에서도 Locale 을 추가해서 GroupingSeparator 와 DecimalSeparator 을 ,. 고정이 아닌 Locale 에 맞게 바뀌도록 해야 한다.

```kotlin
override fun afterTextChanged(s: Editable?) {
    if (busy) return

    if (s.isNullOrBlank()) {
        onDecimalChanged?.invoke(BigDecimal.ZERO)
        return
    }

    busy = true

    val (groupedDecimal, decimal) = stringToDecimal(
        s.toString(),
        maxIntPart,
        maxFractionPart,
        maxFractionDigits,
        onDecimalChanged != null
    )
    s.replace(0, s.length, groupedDecimal)
    decimal?.let {
        onDecimalChanged?.invoke(it)
    }
    busy = false
}
...
private fun stringToDecimal(
        decimalString: String,
        maxIntPart: CharSequence?,
        maxFractionPart: CharSequence?,
        maxFractionDigits: Int,
        parseBigDecimal: Boolean
    ): Pair<CharSequence, BigDecimal?> {
        var intPart: CharSequence
        var fractionPart: CharSequence
        var hasDecimalSeparator: Boolean
    	//decimal separator 가 없다면 정수부만 있음.
        if (decimalString.contains(decimalSeparator)) {
            decimalString.split(decimalSeparator).take(2).let { array ->
                intPart = array[0]
                fractionPart = array[1]
            }
            hasDecimalSeparator = true
        } else {
            intPart = decimalString
            fractionPart = ""
            hasDecimalSeparator = false
        }
        //정수부에 있는 group separator 를 제거하기 위함.
        intPart = intPart.replace(numberFilter, "").let {
            naturalNumberRegex.find(it)?.value ?: "0"
        }
        fractionPart = fractionNumberRegex.find(fractionPart)?.value ?: ""

        if (maxIntPart != null && maxFractionPart != null && isBiggerThenMax(
                intPart,
                fractionPart,
                maxIntPart,
                maxFractionPart
            )
        ) {
            intPart = maxIntPart
            fractionPart = maxFractionPart
            hasDecimalSeparator = maxFractionPart.isNotEmpty()
        }

        if (maxFractionDigits == 0) {
            //정수만 허용
            hasDecimalSeparator = false
            fractionPart = ""
        } else if (fractionPart.isNotBlank() && fractionPart.length > maxFractionDigits) {
            //소수부 제한이 있을 경우 소수부 절삭.
            fractionPart = fractionPart.subSequence(0, fractionPart.length)
        }

        val formattedDecimalString = getFormattedDecimalString(intPart, fractionPart, hasDecimalSeparator)
        val bigDecimal = if (parseBigDecimal) getBigDecimal(intPart, fractionPart) else null
        return formattedDecimalString to bigDecimal

    }

	//decimal separator 을 쓰지 않기 위해 movePointLeft 사용.
    private fun getBigDecimal(intPart: CharSequence, fractionPart: CharSequence): BigDecimal {
        val plainString = StringBuilder(intPart).append(fractionPart).toString()
        return BigDecimal(plainString).movePointLeft(fractionPart.length)
    }

    /**
     * @param intPart 정수부
     * @param fractionPart 소수부 (fractionDigits 적용되어 있음)
     * @param hasDecimalSeparator 소수점 여부 (10.) 입력을 허용하기 위함.
     * @return decimalFormat 이 적용된 CharSequence
     * ex)  intPart: 1234567890
     *      fractionPart: 1234567890
     *      return: 1,234,567,890.1234567890
     */
    private fun getFormattedDecimalString(
        intPart: CharSequence,
        fractionPart: CharSequence,
        hasDecimalSeparator: Boolean
    ): CharSequence {
        val formattedDecimalBuilder = StringBuilder()
        /**
         * int part
         *
         * 정수 부를 뒤집고 뒤에서부터 3자리 마다 ,를 찍은 후 다시 뒤집음.
         * index    012 345 6
         * reversed 432,112,3
         */
        intPart.reversed().forEachIndexed { index, c ->
            if (index.rem(groupSize) == 0 && index > 0) {
                formattedDecimalBuilder.append(groupingSeparator)
            }
            formattedDecimalBuilder.append(c)
        }
        formattedDecimalBuilder.reverse()

        /**
         * fraction part
         */
        if (hasDecimalSeparator) {
            formattedDecimalBuilder.append(decimalSeparator)
            formattedDecimalBuilder.append(fractionPart)
        }
        return formattedDecimalBuilder
    }

```

  locale 을 고려하지 않을때는 정규표현식을 이용해서 숫자를 쉽게 추출할 수 있지만 locale이 포함되면 불편해진다. grouping separator 혹은 decimal separator 이 정규표현식에서 특수문자로 쓰일수도 있기 때문이다. 그래서 정규표현식보단 decimal separator 로 split 해서 정수부/소수부를 나눈 후 숫자만 추출하는게 좋다.  그 후에는 정수부에 숫자를 제외한 부분을 지우고 grouping separator 를 추가한 후 정수부와 decimal separator, 소수부를 합치면 된다.

---

샘플 코드 <https://github.com/minchulking91/DecimalFormatSample>

