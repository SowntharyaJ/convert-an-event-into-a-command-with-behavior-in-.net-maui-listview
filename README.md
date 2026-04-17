# convert-an-event-into-a-command-with-behavior-in-.net-maui-listview

This repository contains a sample demonstrating how to convert an event into a c0mmand using behaviors to follow the MVVM pattern in .NET MAUI ListView (SfListView).

## Sample

```xaml
<syncfusion:SfListView
            x:Name="listView"
            ItemSize="75"
            ItemsSource="{Binding ContactsInfo}"
            SelectionMode="Single">
            <syncfusion:SfListView.Behaviors>
                <local:EventToCommandBehavior Command="{Binding TapCommand}" EventName="ItemTapped" />
            </syncfusion:SfListView.Behaviors>

    <syncfusion:SfListView.ItemTemplate>
        <DataTemplate>
            <Grid VerticalOptions="Center">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto" />
                    <RowDefinition Height="1" />
                </Grid.RowDefinitions>
                <Grid Grid.Row="0">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="50" />
                        <ColumnDefinition Width="*" />
                    </Grid.ColumnDefinitions>
                    <Image
                        Grid.Column="0"
                        HeightRequest="50"
                        Source="{Binding ContactImage}"
                        WidthRequest="50" />
                    <StackLayout
                        Grid.Column="1"
                        HorizontalOptions="StartAndExpand"
                        Orientation="Vertical"
                        VerticalOptions="Center">
                        <Label
                            HorizontalOptions="Center"
                            HorizontalTextAlignment="Center"
                            LineBreakMode="WordWrap"
                            Text="{Binding ContactName}"
                            TextColor="#474747"
                            VerticalOptions="Center"
                            VerticalTextAlignment="Center" />
                        <Label
                            HorizontalOptions="Center"
                            HorizontalTextAlignment="Center"
                            LineBreakMode="WordWrap"
                            Text="{Binding ContactNumber}"
                            TextColor="#474747"
                            VerticalOptions="Center"
                            VerticalTextAlignment="Center" />
                    </StackLayout>
                </Grid>
                <BoxView Grid.Row="1" />
            </Grid>
        </DataTemplate>
    </syncfusion:SfListView.ItemTemplate>
</syncfusion:SfListView>
```

```c#
public class EventToCommandBehavior : BehaviorBase<SfListView>
{
    Delegate eventHandler;

    public static readonly BindableProperty EventNameProperty = BindableProperty.Create ("EventName", typeof(string), typeof(EventToCommandBehavior), null, propertyChanged: OnEventNameChanged);
    public static readonly BindableProperty CommandProperty = BindableProperty.Create ("Command", typeof(ICommand), typeof(EventToCommandBehavior), null);
    public static readonly BindableProperty CommandParameterProperty = BindableProperty.Create ("CommandParameter", typeof(object), typeof(EventToCommandBehavior), null);
    public static readonly BindableProperty InputConverterProperty = BindableProperty.Create ("Converter", typeof(IValueConverter), typeof(EventToCommandBehavior), null);

    public string EventName {
        get { return (string)GetValue (EventNameProperty); }
        set { SetValue (EventNameProperty, value); }
    }

    public ICommand Command {
        get { return (ICommand)GetValue (CommandProperty); }
        set { SetValue (CommandProperty, value); }
    }

    public object CommandParameter {
        get { return GetValue (CommandParameterProperty); }
        set { SetValue (CommandParameterProperty, value); }
    }

    public IValueConverter Converter {
        get { return (IValueConverter)GetValue (InputConverterProperty); }
        set { SetValue (InputConverterProperty, value); }
    }

    protected override void OnAttachedTo(SfListView bindable)
    {
        base.OnAttachedTo(bindable);
        RegisterEvent(EventName);
    }
    protected override void OnDetachingFrom(SfListView bindable)
    {
        base.OnDetachingFrom(bindable);
        DeregisterEvent(EventName);
    }

    void RegisterEvent (string name)
    {
        if (string.IsNullOrWhiteSpace (name)) {
            return;
        }

        EventInfo eventInfo = AssociatedObject.GetType().GetRuntimeEvent(name);
        if (eventInfo == null) {
            throw new ArgumentException (string.Format ("EventToCommandBehavior: Can't register the '{0}' event.", EventName));
        }
        MethodInfo methodInfo = typeof(EventToCommandBehavior).GetTypeInfo ().GetDeclaredMethod ("OnEvent");
        eventHandler = methodInfo.CreateDelegate (eventInfo.EventHandlerType, this);
        eventInfo.AddEventHandler (AssociatedObject, eventHandler);
    }

    void DeregisterEvent (string name)
    {
        if (string.IsNullOrWhiteSpace (name)) {
            return;
        }

        if (eventHandler == null) {
            return;
        }
        EventInfo eventInfo = AssociatedObject.GetType ().GetRuntimeEvent (name);
        if (eventInfo == null) {
            throw new ArgumentException (string.Format ("EventToCommandBehavior: Can't de-register the '{0}' event.", EventName));
        }
        eventInfo.RemoveEventHandler (AssociatedObject, eventHandler);
        eventHandler = null;
    }

    void OnEvent (object sender, object eventArgs)
    {
        if (Command == null) {
            return;
        }

        object resolvedParameter;
        if (CommandParameter != null) {
            resolvedParameter = CommandParameter;
        } else if (Converter != null) {
            resolvedParameter = Converter.Convert (eventArgs, typeof(object), AssociatedObject, null);
        } else {
            resolvedParameter = eventArgs;
        }		

        if (Command.CanExecute (resolvedParameter)) {
            Command.Execute (resolvedParameter);
        }
    }

    static void OnEventNameChanged (BindableObject bindable, object oldValue, object newValue)
    {
        var behavior = (EventToCommandBehavior)bindable;
        if (behavior.AssociatedObject == null) {
            return;
        }

        string oldEventName = (string)oldValue;
        string newEventName = (string)newValue;

        behavior.DeregisterEvent (oldEventName);
        behavior.RegisterEvent (newEventName);
    }
}
```

## Requirements to run the demo

* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/) or [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/)
* Xamarin add-ons for Visual Studio (available via the Visual Studio installer).

## Troubleshooting

### Path too long exception

If you are facing path too long exception when building this example project, close Visual Studio and rename the repository to short and build the project.
