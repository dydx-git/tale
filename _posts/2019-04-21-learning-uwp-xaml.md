---
layout: post
title: "Learning UWP XAML"
categories: Code
author: "Muhammad Taha"
---

## UWP XAML

### Formatting text:
`<Run>` is an inline TextBlock Property and it can be used to format plain texts within the textblock like so:
{% highlight xml %}
<TextBlock Name="myText" TextWrapping="Wrap">
    This plain text is
    <Run FontStyle="Italic" Text="now formatted"/>
</TextBlock>
{% endhighlight %}

### Resources:
Resources limited to a specific page should go just before defining the main Grid/StackPanel like so:
{% highlight xml %}
<Page.Resources>
    <Style TargetType="ControlName">
        <Setter Property="PropertyName" Value="PropertyValue" />
        <Setter Property="AnotherPropertyName" Value="AnotherPropertyValue" />
    </Style>
</Page.Resources>
{% endhighlight %}
Resources can also be created inside certain context like a StackPanel/Grid by using `Grid.Resources`. Resources can be created for XAML elements as well as any other control that needs to be reused.
