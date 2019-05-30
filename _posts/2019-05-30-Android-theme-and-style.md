---
title: "안드로이드 테마와 스타일을 언제 써야 할까?"
date: 2019-05-30 22:00:00 +0900
categories: android theme style
---

# 안드로이드 테마와 스타일을 언제 써야 할까? 

  InputTextLayout 에 디자인을 적용하다가 InputTextLayout 은 단순히 style 만 가지고는 원하는 대로 디자인을 할 수 없다. 이를 위해 테마와 스타일에 관한 많은 글을 찾아봤지만 잘못 설명 되어 있는 글들이 많고 style 의 경우 resource definition 을 찾아 들어가는 탐색을 하려고 해도 테마간 구조가 복잡하게 엮여있어 탐색이 힘들기 때문에 공식 문서를 통해 정리를 해보려 한다.

  많은 글들에서 style attribute (style="@style/...") 로 적용 할 수 있는 attribute 와 theme attribute(android:theme="@style/...") 로 적용 할 수 있는 attribute 가 따로 있다고 말하지만 이건 완벽하게 맞는 설명은 아니다. 경우에 따라 같은 style 을 style attribute 로 적용할 경우와 theme attribute 로 적용할 경우 결과가 달라지긴 하지만 기본적으로 style 이 가지고 있는 속성들을 적용한다. 공식 문서에는 다음과 같이 나와있다.

> Each attribute specified in the style is applied to that view if the view accepts it. The view simply ignores any attributes that it does not accept.
>
> Note: Only the element to which you add the style attribute receives those style attributes—any child views do not apply the styles. If you want child views to inherit styles, instead apply the style with the android:theme attribute.
>
> However, instead of applying a style to individual views, you'll usually apply styles as a theme for your entire app, activity, or collection of views.

* 스타일에 있는 속성들은 스타일이 적용된 뷰에 적용됨. 만약 적용할 수 없다면 무시함.

* 스타일의 경우 적용된 뷰의 자식 뷰들은 스타일이 적용되지 않음. 만약 자식까지 같은 스타일을 적용하고 싶다면 테마를 사용할 것.

* 개별적으로 뷰에 스타일을 적용하는 것 대신에 App 또는 Activity, ViewGroup 에 테마를 적용 할 수 있음. 

> Now every view in the app or activity applies the styles defined in the given theme. If a view supports only some of the attributes declared in the style, then it applies only those attributes and ignores the ones it does not support.

* 테마에 textViewStyle, buttonStyle 등 스타일을 지정하면 해당 테마에 있는 textView 와 button 에 해당 스타일을 사용한다.

## 커스텀 스타일과 확장.

```xml
<!-- AppTheme 는 Theme.AppCompat.Light.DarkActionBar 를 포함함 -->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
</style>

<!-- AppTheme.NoActionBar 는 AppTheme 를 포함함 -->
<style name="AppTheme.NoActionBar">
    <item name="windowActionBar">false</item>
    <item name="windowNoTitle">true</item>
</style>

<!-- AppTheme.NoActionBar2 는 AppTheme 를 포함하지 않음. (parent 와 dot 을 같이 쓸 경우. parent 만 적용.) -->
<style name="AppTheme.NoActionBar2" parent="Theme.AppCompat.Light.NoActionBar">
    
</style>
```



## ThemeOverlay 는 언제 써야 하는지.

  ThemeOverlay 의 경우. 현재 적용되어 있는 테마에서 일부 속성을 가져오고, 일부 속성을 새로 정의한다. 우리가 popup theme 등에 ThemeOverlay 를 상속한 테마를 쓰는 이유는 대부분의 popup 들은 primary, secondary color 등 서비스의 브랜딩에 쓰이는 색은 그대로 사용하고 배경색과 텍스트 색만 변경해서 쓰기 때문이다. 

AppCompat 라이브러리를 뜯어보면 해당 구조를 파악 할 수 있다.

```xml
<!-- values.xml -->
<style name="ThemeOverlay.AppCompat.Light" parent="Base.ThemeOverlay.AppCompat.Light"/>

<style name="Base.ThemeOverlay.AppCompat.Light" parent="Platform.ThemeOverlay.AppCompat.Light">
    <item name="android:windowBackground">@color/background_material_light</item>
    <item name="android:colorForeground">@color/foreground_material_light</item>
    <item name="android:colorForegroundInverse">@color/foreground_material_dark</item>
    <item name="android:colorBackground">@color/background_material_light</item>
    <item name="android:colorBackgroundCacheHint">@color/abc_background_cache_hint_selector_material_light</item>
    <item name="colorBackgroundFloating">@color/background_floating_material_light</item>

    <item name="android:textColorPrimary">@color/abc_primary_text_material_light</item>
    <item name="android:textColorPrimaryInverse">@color/abc_primary_text_material_dark</item>
    <item name="android:textColorSecondary">@color/abc_secondary_text_material_light</item>
    <item name="android:textColorSecondaryInverse">@color/abc_secondary_text_material_dark</item>
    <item name="android:textColorTertiary">@color/abc_secondary_text_material_light</item>
    <item name="android:textColorTertiaryInverse">@color/abc_secondary_text_material_dark</item>
    <item name="android:textColorPrimaryDisableOnly">@color/abc_primary_text_disable_only_material_light</item>
    <item name="android:textColorPrimaryInverseDisableOnly">@color/abc_primary_text_disable_only_material_dark</item>
    <item name="android:textColorHint">@color/abc_hint_foreground_material_light</item>
    <item name="android:textColorHintInverse">@color/abc_hint_foreground_material_dark</item>
    <item name="android:textColorHighlight">@color/highlighted_text_material_light</item>

    <item name="colorControlNormal">?android:attr/textColorSecondary</item>
    <item name="colorControlHighlight">@color/ripple_material_light</item>
    <item name="colorButtonNormal">@color/button_material_light</item>
    <item name="colorSwitchThumbNormal">@color/switch_thumb_material_light</item>

    <!-- Used by MediaRouter -->
    <item name="isLightTheme">true</item>
</style>

<!-- values-21.xml -->
<style name="Platform.ThemeOverlay.AppCompat" parent="">
    <!-- Copy our color theme attributes to the framework -->
    <item name="android:colorPrimary">?attr/colorPrimary</item>
    <item name="android:colorPrimaryDark">?attr/colorPrimaryDark</item>
    <item name="android:colorAccent">?attr/colorAccent</item>
    <item name="android:colorControlNormal">?attr/colorControlNormal</item>
    <item name="android:colorControlActivated">?attr/colorControlActivated</item>
    <item name="android:colorControlHighlight">?attr/colorControlHighlight</item>
    <item name="android:colorButtonNormal">?attr/colorButtonNormal</item>
</style>
```

![theme_inheritance](https://user-images.githubusercontent.com/8017683/58634652-c1818e80-8326-11e9-849e-6e95387f7c52.png)



## ```android``` namespace 가 있는 것과 없는 것의 차이.

android:colorControlNormal 과 colorControlNormal의 차이는 무엇일까?

![android_framework_color_control_normal](https://user-images.githubusercontent.com/8017683/58634649-bf1f3480-8326-11e9-8ca2-be9d703dd49e.png)

android:colorControlNormal 는 Android SDK platforms에 선언되어 있다.

Android Framework 에 있는 속성의 경우 ```android``` prefix 사용해야 한다.



![appcompat_color_control_normal](https://user-images.githubusercontent.com/8017683/58634649-bf1f3480-8326-11e9-8ca2-be9d703dd49e.png)

colorControlNormal 은 SupportLibrary 에 선언되어 있다.

## 테마와 스타일 적용 방법 예시

스타일을 적용할 때는 우선순위를 잘 알아야 한다. 공식 문서에 나와있는 우선순위는 다음과 같다.

> When choosing how to style your app, be mindful of Android's style hierarchy. In general, you should use themes and styles as much as possible for consistency. If you've specified the same attributes in multiple places, the list below determines which attributes are ultimately applied. The list is ordered ㅂfrom highest precedence to lowest:
>
> 1. Applying character- or paragraph-level styling via text spans to `TextView`-derived classes
> 2. Applying attributes programmatically
> 3. Applying individual attributes directly to a View
> 4. Applying a style to a View
> 5. Default styling
> 6. Applying a theme to a collection of Views, an activity, or your entire app
> 7. Applying certain View-specific styling, such as setting a `TextAppearance` on a `TextView`

1번의 경우 SpannableString 등으로 character 나 paragraph 단위에서 지정한 스타일이다.
2번의 경우 코드를 통해 지정한 속성. 이런 경우 이미 View는 xml 에서 inflate 되었으니 나중에 지정한게 우선순위를 가지는게 자연스럽다.
3번의 경우 xml 에서 개별적인 뷰에 지정한 속성이다.
4번의 경우는 스타일을 통해서 지정된 속성이다.
5번의 경우는 theme 에 지정된 textViewStyle, buttonStyle 같은 기본 스타일을 말한다.
6번의 경우 상위 뷰 혹은 액티비티, 어플리케이션의 테마에 지정된 속성이다. (상위 뷰가 여러 개 인데 각각 다른 테마를 가지고 있다면 최상위에 있는 테마부터 적용이 된다. 테마는 덮어씌워지므로 중복되는 속성이 있을 경우 최하위 테마가 남는다.)
7번의 경우는 특정 뷰에만 지정되는 스타일이다. textAppearane 같은 것들이 있다.

6번과 7번이 조금 어색할 수 있는데 예시를 보면 쉽다.

```xml
<resources>

    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Application 에는 AppTheme 가 적용되어 있다. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
        <item name="android:textViewStyle">@style/MyTextViewStyle</item>
        <item name="android:text">AppTheme</item>
    </style>

    <style name="MyTextViewStyle">
        <item name="android:text">MyTextViewStyle</item>
    </style>
</resources>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="16dp"
        tools:context=".MainActivity">

    <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
</LinearLayout>
```

  텍스트뷰는 가장 먼저 6번에 따라 AppTheme 가 가지고 있는 android:text 속성을 받는다. 그 후에 5번에 따라 MyTextViewStyle 에 있는 속성이 적용되는데 스타일 또한 android:text 속성을 가지기 때문에 결과적으로 텍스트뷰에는 "MyTextViewStyle" 가 찍히게 된다.

```xml
<resources>

    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Application 에는 AppTheme 가 적용되어 있다. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
        
        <!-- 5번째 우선순위 -->
        <!--<item name="android:textViewStyle">@style/MyTextViewStyle</item>-->
        
        <!-- 6번째 우선순위 -->
        <!--<item name="android:textColor">#00F</item>-->
    </style>

    <style name="MyTextViewStyle">
        <item name="android:textColor">#0F0</item>
    </style>

    <style name="MyTextViewStyle2">
        <item name="android:textColor">#F00</item>
    </style>
</resources>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="16dp"
        tools:context=".MainActivity">

    <!-- android:textAppearance 는 7번째 우선순위를 가진다. -->
    <TextView
            android:text="Test"
            android:textAppearance="@style/MyTextViewStyle2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
</LinearLayout>
```

  7번의 경우는 잘못 쓸 경우 3번과 헷갈리기 때문에 이 내용을 모르면 오류를 찾기가 쉽지 않다. 텍스트뷰에 직접 textAppearance 를 넣기 때문에 Theme 로 적용된 textColor 나 textViewStyle 보다 우선순위를 가질것으로 보이지만 그렇지 않다. theme 나 style에 TextAppearance 와 겹치는 항목 (textColor, textSize 등) 이 있다면 개별 뷰에 적용한 textAppearance 는 무조건 무시되기 때문에 주의가 필요하다.  그래서 글로벌로 텍스트 색상이나 텍스트 사이즈를 변경 하고 싶을 경우 다음과 같이 하는게 더 좋다.

```xml
<resources>

    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
        <item name="android:textViewStyle">@style/Widget.AppCompat.TextView</item>
        <!--<item name="android:textColor">#00F</item>-->
    </style>

    <style name="MyTextViewStyle">
        <item name="android:textAppearance">@style/MyTextViewStyle3</item>
    </style>

    <style name="MyTextViewStyle2">
        <item name="android:textColor">#F00</item>
    </style>

    <style name="MyTextViewStyle3">
        <item name="android:textColor">#FF0</item>
    </style>
</resources>
```

  이럴 경우 TextView 에 textAppearance 속성이 없을 경우 MyTextViewStyle3 가 적용되고, 있을 경우엔 지정한 textAppearance 가 적용된다.

  다음 포스팅에서는 안드로이드 SDK 혹은 지원 라이브러리에서 제공하는 테마를 재사용 하는 방법에 대해서 알아보자.

---

## 참고문헌

<https://developer.android.com/guide/topics/ui/look-and-feel/themes>

