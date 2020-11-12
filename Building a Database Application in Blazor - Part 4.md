# Building a Database Application in Blazor 
## Part 4 - UI Components

## Introduction

This is the fourth article in the series looking at how to build and structure a real Database Application in Blazor. The articles so far are:

1. [Project Structure and Framework](https://www.codeproject.com/Articles/5279560/Building-a-Database-Application-in-Blazor-Part-1-P)
2. [Services - Building the CRUD Data Layers](https://www.codeproject.com/Articles/5279596/Building-a-Database-Application-in-Blazor-Part-2-S)
3. [View Components - CRUD Edit and View Operations in the UI](https://www.codeproject.com/Articles/5279963/Building-a-Database-Application-in-Blazor-Part-3-C)
4. UI Components - Building HTML/CSS Controls

Further articles will look at 
* List Operations in the UI
* A walk through detailing how to add more records to the application - in this case weather stations and weather station data.

This article looks at the components we use in the UI and then focuses on how to build generic UI Components from HTML and CSS.

### Sample Project and Code

All the sample code and libraries are on GitHub - [CEC.Blazor GitHub Repository](https://github.com/ShaunCurtis/CEC.Blazor).

### Components

For a detailed look at components read my article [A Dive into Blazor Components](https://www.codeproject.com/Articles/5277618/A-Dive-into-Blazor-Components).

To summarise, everything in the Blazor, other than the start page is a component.

I divide components into four categories:
1. Views - these are routed components/views.
2. Forms - there can be one or more forms within a View, and one or more UIControls within a form.  Edit/Display/List components are all forms.
3. UIControls - these are collections of HTML markup and CSS. Similar to Form Controls.
4. Layouts - these a special components used to layout a View.

### Views

Views are specific to the application and live in the *Routes* folder.

The Weather Forecast Viewer and List Views are shown below.
```cs
// CEC.Blazor.Server/Routes/WeatherForecastViewerView.cs
@page "/WeatherForecast/View"

@namespace CEC.Blazor.Server.Pages

@inherits ApplicationComponentBase

<WeatherViewer></WeatherViewer>
```
The list view defines a UIOptions object that control various list control display options.
```cs
// CEC.Blazor.Server/Routes/WeatherForecastListView.cs
@page "/WeatherForecast"

@layout MainLayout

@namespace CEC.Blazor.Server.Routes

@inherits ApplicationComponentBase

<WeatherList UIOptions="this.UIOptions" ></WeatherList>

@code {
    public UIOptions UIOptions => new UIOptions()
    {
        ListNavigationToViewer = true,
        ShowButtons = true,
        ShowAdd = true,
        ShowEdit = true
    };
}
```

### Forms

Forms are also project specific, but are common to both WASM and Server deployments.  In the Weather Application they reside in the CEC.Weather library.  

The code below shows the Weather Viewer.  It's all UI Controls, no HTML markup.  The markup lives inside the controls - we'll look at some example UI Controls later.

```html
// CEC.Weather/Components/Forms/WeatherForecastViewerForm.razor
<UICard>
    <Header>
        @this.PageTitle
    </Header>
    <Body>
        <UIErrorHandler IsError="this.IsError" IsLoading="this.IsDataLoading" ErrorMessage="@this.RecordErrorMessage">
            <UIContainer>
                <UIRow>
                    <UILabelColumn Columns="2">
                        Date
                    </UILabelColumn>
                    <UIColumn Columns="2">
                        <FormControlPlainText Value="@this.Service.Record.Date.AsShortDate()"></FormControlPlainText>
                    </UIColumn>
                    <UILabelColumn Columns="2">
                        ID
                    </UILabelColumn>
                    <UIColumn Columns="2">
                        <FormControlPlainText Value="@this.Service.Record.ID.ToString()"></FormControlPlainText>
                    </UIColumn>
                    <UILabelColumn Columns="2">
                        Frost
                    </UILabelColumn>
                    <UIColumn Columns="2">
                        <FormControlPlainText Value="@this.Service.Record.Frost.AsYesNo()"></FormControlPlainText>
                    </UIColumn>
                </UIRow>
            ..........
            </UIContainer>
        </UIErrorHandler>
        <UIContainer>
            <UIRow>
                <UIColumn Columns="6">
                    <UIButton Show="this.IsLoaded" ColourCode="Bootstrap.ColourCode.dark" ClickEvent="(e => this.NextRecord(-1))">
                        Previous
                    </UIButton>
                    <UIButton Show="this.IsLoaded" ColourCode="Bootstrap.ColourCode.dark" ClickEvent="(e => this.NextRecord(1))">
                        Next
                    </UIButton>
                </UIColumn>
                <UIButtonColumn Columns="6">
                    <UIButton Show="!this.IsModal" ColourCode="Bootstrap.ColourCode.nav" ClickEvent="(e => this.NavigateTo(PageExitType.ExitToList))">
                        Exit To List
                    </UIButton>
                    <UIButton Show="!this.IsModal" ColourCode="Bootstrap.ColourCode.nav" ClickEvent="(e => this.NavigateTo(PageExitType.ExitToLast))">
                        Exit
                    </UIButton>
                    <UIButton Show="this.IsModal" ColourCode="Bootstrap.ColourCode.nav" ClickEvent="(e => this.ModalExit())">
                        Exit
                    </UIButton>
                </UIButtonColumn>
            </UIRow>
        </UIContainer>
    </Body>
```

The code behind page is relatively simple - the complexity is in the boilerplate code in parent classes.  It loads the record specific Controller service.

```C#
// CEC.Weather/Components/Forms/WeatherForecastViewerForm.razor.cs
public partial class WeatherViewer : RecordComponentBase<DbWeatherForecast>
{
    public partial class WeatherViewer : RecordComponentBase<DbWeatherForecast, WeatherForecastDbContext>
    {
        [Inject]
        private WeatherForecastControllerService ControllerService { get; set; }

        public override string PageTitle => $"Weather Forecast Viewer {this.Service?.Record?.Date.AsShortDate() ?? string.Empty}".Trim();

        protected async override Task OnInitializedAsync()
        {
            this.Service = this.ControllerService;
            await base.OnInitializedAsync();
        }

        //  example code to show jumping records with querystring changes
        protected void NextRecord(int increment) 
        {
            var rec = (this._ID + increment) == 0 ? 1 : this._ID + increment;
            rec = rec > this.Service.BaseRecordCount ? this.Service.BaseRecordCount : rec;
            this.NavManager.NavigateTo($"/WeatherForecast/View?id={rec}");
        }
    }
}
```

### UI Controls

The application uses UI Controls to separate HTML and CSS markup from Views and Forms.  Bootstrap is used as the UI Framework.

##### UIBase

All library UI Controls inherit from *UIBase*.  This implements *IComponent*, we don't use *ComponentBase* because we don't need it's complexity.   The complete class is too long to show - you can view it [here](https://github.com/ShaunCurtis/CEC.Blazor/blob/master/CEC.Blazor/Components/UIControls/UI/UIBase.cs).

It builds an HTML DIV block that you can turn on or off.

*IComponent* looks like this.  

```cs
public interface IComponent
{
    /// Called by the builder to attach it to a render tree
    void Attach(RenderHandle renderHandle);
    /// Called by the render tree whenever the component parameters are changed
    Task SetParametersAsync(ParameterView parameters);
}
```
*Attach* is called whenever a render tree is built by the RenderTreeBuilder and attaches the component to the render tree.  We assign *renderHandle* to a class property *_renderHhandle*.  The *RenderHandle* class gives us access to the RenderQueue on the RenderTree through *RenderHandle.Render(RenderFragment)*.  

RenderFragment is a delegate which is run by the RenderQueue executer.  We build the RenderFragment property *_componentRenderFragment* in the class initialisation method.  You can see it below. It sets the *_RenderEventQueued* property to false as it's now being run, and calls *BuildRenderTree* which builds the components markup code.

*StateHasChanged* is our method for triggering a UI update.  It sets *_RenderEventQueued* property to true and loads *_componentRenderFragment* into the Render Tree RenderQueue.

```cs
// Code blocks from CEC.Blazor/Components/UIControls/UIBase.cs

// RenderHandle assigned in Attack
private RenderHandle _renderHandle;

/// Render Fragment to render this object
private readonly RenderFragment _componentRenderFragment;

/// Boolean Flag to track if there's a pending render event queued
private bool _RenderEventQueued;

//  Class initialisation - we build the RenderFragement _componentRenderFragment
public UIBase() => _componentRenderFragment = builder =>
{
    this._RenderEventQueued = false;
    BuildRenderTree(builder);
};

//  IComponent Interface Method - called by the RenderTree Builder when it attaches the component to a Render Tree
public void Attach(RenderHandle renderHandle) => _renderHandle = renderHandle;

// Method to kick off a re-render of the component
public void StateHasChanged()
{
    // Check if we already have a render queued - if so then it will handle the changes
    if (!this._RenderEventQueued)
    {
        // Flag so we know we have a render queued
        this._RenderEventQueued = true;
        // Load the Render Fragment into the Render Queue
        _renderHandle.Render(_componentRenderFragment);
    }
}

```
Whenever the component's Parameters are changed in the Render Tree, *SetParametersAsync* is called.  In *UIBase* we have a simple rendered component so we update the component parameters by calling *SetParameterProperties(this)* and then re-render the component by calling *StateHasChanged*  

```cs
public virtual Task SetParametersAsync(ParameterView parameters)
{
    parameters.SetParameterProperties(this);
    StateHasChanged();
    return Task.CompletedTask;
}
```
The rest of *UIBase* is shown below with detailed commenting.

```cs
/// Gets the additional attributes that have been applied to the Component in Markup.
[Parameter(CaptureUnmatchedValues = true)] public IDictionary<string, object> AdditionalAttributes { get; set; }

/// The content between the opening and closing tags
[Parameter]
public RenderFragment ChildContent { get; set; }

/// The component Tag that can be set - default is a DIV
[Parameter]
public virtual string Tag { get; set; } = "div";

/// the Tag actually used in BuildRenderTree.  Can be overridden in child components
// so protected overridden string _Tag => "span"; sets the Tag as SPAN
protected virtual string _Tag => this.Tag;

/// Css for component that can be set
[Parameter]
public virtual string Css { get; set; } = string.Empty;

/// <summary>
/// Additional Css that is appended to the end of the base Css
/// </summary>
[Parameter]
public string AddOnCss { get; set; } = string.Empty;

/// Property for fixing the base Css.  Base returns the Parameter Css, but can be overridden in inherited classes
protected virtual string _BaseCss => this.Css;

/// Property for fixing the Add On Css.  Base returns the Parameter AddOnCss, but can be overridden say to String.Empty in inherited classes.  You can set this to String.Empty to stop any Css being added
protected virtual string _AddOnCss => this.AddOnCss;

/// <summary>
/// Actual calculated Css string used in the component.  CleanUpCss gets ride of double spaces, etc.
/// </summary>
protected virtual string _Css => this.CleanUpCss($"{this._BaseCss} {this._AddOnCss}");

/// Method to clean up the Css String
protected string CleanUpCss(string css)
{
    while (css.Contains("  ")) css = css.Replace("  ", " ");
    return css.Trim();
}
/// Boolean property that dictates if the componet is rendered
[Parameter]
public virtual bool Show { get; set; } = true;

/// Actual Show used in  the RenderTreeBuilder.  Can be overridden
protected virtual bool _Show => this.Show;

/// Property to allow the component content to be overridden.  Set to a value and it will be used instead of any child content
protected virtual string _Content => string.Empty;

/// List of Attributes to trim from AdditionalAttributes - the default is "class" as this is set by the component and we want to ignore it. 
protected List<string> UsedAttributes { get; set; } = new List<string>() { "class" };

/// inherited BuildRenderTree
protected virtual void BuildRenderTree(RenderTreeBuilder builder)
{
    // Checks if we need to show the component.  If not nothing is rendered
    if (this._Show)
    {
        // clean out all the Duplicate Attributes
        this.ClearDuplicateAttributes();
        // Open the first element
        builder.OpenElement(0, this._Tag);
        // Add the AdditionAttributes
        builder.AddMultipleAttributes(1, AdditionalAttributes);
        // Add the Css
        builder.AddAttribute(2, "class", this._Css);
        // Check if we have overidden content, if so use it
        if (!string.IsNullOrEmpty(this._Content)) builder.AddContent(3, (MarkupString)this._Content);
        // Otherwise add the child content
        else if (this.ChildContent != null) builder.AddContent(3, ChildContent);
        // Close the element
        builder.CloseElement();
    }
}

/// Method to clean up the Additional Attributes
protected void ClearDuplicateAttributes()
{
    if (this.AdditionalAttributes != null && this.UsedAttributes != null)
    {
        foreach (var item in this.UsedAttributes)
        {
            if (this.AdditionalAttributes.ContainsKey(item)) this.AdditionalAttributes.Remove(item);
        }
    }
}
```

##### UIBootstrapBase

*UIBootstrapBase* adds extra functionality for Bootstrap components. Formatting options such a component colour and sizing are represented as Enums, and Css fragments built based on the selected Enum.

```c#
// CEC.Blazor/Components/UIControls/UIBootstrapBase.cs
public class UIBootstrapBase : UIBase
{
    protected virtual string CssName { get; set; } = string.Empty;

    /// Bootstrap Colour for the Component
    [Parameter]
    public Bootstrap.ColourCode ColourCode { get; set; } = Bootstrap.ColourCode.info;

    /// Bootstrap Size for the Component
    [Parameter]
    public Bootstrap.SizeCode SizeCode { get; set; } = Bootstrap.SizeCode.normal;

    /// Property to set the HTML value if appropriate
    [Parameter]
    public string Value { get; set; } = "";

    /// Property to get the Colour CSS
    protected virtual string ColourCssFragment => GetCssFragment<Bootstrap.ColourCode>(this.ColourCode);

    /// Property to get the Size CSS
    protected virtual string SizeCssFragment => GetCssFragment<Bootstrap.SizeCode>(this.SizeCode);

    /// CSS override
    protected override string _Css => this.CleanUpCss($"{this.CssName} {this.SizeCssFragment} {this.ColourCssFragment} {this.AddOnCss}");

    /// Method to format as Bootstrap CSS Fragment
    protected string GetCssFragment<T>(T code) => $"{this.CssName}-{Enum.GetName(typeof(T), code).Replace("_", "-")}";
}
```
### Some Examples

##### UIButton

This is a standard Bootstrap Button. 
1. *ButtonType* and *ClickEvent* are specific to buttons.
2. *CssName* and *_Tag* are hardwired.
3. *ButtonClick* handles the button click event.
4. *BuildRenderTree* builds the markup and wires the JSInterop *onclick* event.
5. *Show* controls whether the button gets rendered.

```c#
// CEC.Blazor/Components/UIControls/UIButton.cs
public class UIButton : UIBootstrapBase
{
    /// Property setting the button HTML attribute Type
    [Parameter]
    public string ButtonType { get; set; } = "button";

    /// Override the CssName
    protected override string CssName => "btn";

    /// Override the Tag
    protected override string _Tag => "button";

    /// Callback for a button click event
    [Parameter]
    public EventCallback<MouseEventArgs> ClickEvent { get; set; }

    protected override void BuildRenderTree(RenderTreeBuilder builder)
    {
        if (this.Show)
        {
            builder.OpenElement(0, this._Tag);
            builder.AddAttribute(1, "type", this.ButtonType);
            builder.AddAttribute(2, "class", this._Css);
            builder.AddAttribute(3, "onclick", EventCallback.Factory.Create<MouseEventArgs>(this, this.ButtonClick));
            builder.AddContent(4, ChildContent);
            builder.CloseElement();
        }
    }

    /// Event handler for button click
    protected void ButtonClick(MouseEventArgs e) => this.ClickEvent.InvokeAsync(e);
}
```

Here's some code showing the control in use.

```html
// CEC.Weather/Components/Forms/WeatherViewer.razor
<UIButtonColumn Columns="6">
    <UIButton Show="!this.IsModal" ColourCode="Bootstrap.ColourCode.nav" ClickEvent="(e => this.NavigateTo(PageExitType.ExitToList))">
        Exit To List
    </UIButton>
    <UIButton Show="!this.IsModal" ColourCode="Bootstrap.ColourCode.nav" ClickEvent="(e => this.NavigateTo(PageExitType.ExitToLast))">
        Exit
    </UIButton>
    <UIButton Show="this.IsModal" ColourCode="Bootstrap.ColourCode.nav" ClickEvent="(e => this.ModalExit())">
        Exit
    </UIButton>
</UIButtonColumn>
```

##### UIAlert

This is a standard Bootstrap Alert. 
1. *Alert* is a class to encapsulate an Alert.
2. *ColourCssFragement*, *Show* and *_Content* are wired into the Alert object instance.

```c#
// CEC.Blazor/Components/UIControls/UI/UIAlert.cs
public class UIAlert : UIBootstrapBase
{
    /// Alert to display
    [Parameter]
    public Alert Alert { get; set; } = new Alert();

    /// Set the CssName
    protected override string CssName => "alert";

    /// Property to override the colour CSS
    protected override string ColourCssFragment => this.Alert != null ? GetCssFragment<Bootstrap.ColourCode>(this.Alert.ColourCode) : GetCssFragment<Bootstrap.ColourCode>(this.ColourCode);

    /// Boolean Show override
    protected override bool _Show => this.Alert?.IsAlert ?? false;

    /// Override the content with the alert message
    protected override string _Content => this.Alert?.Message ?? string.Empty;
}
```

Here's some code showing the control in use.

```html
// CEC.Weather/Components/Forms/WeatherEditor.razor
<UIContainer>
    <UIRow>
        <UIColumn Columns="7">
            <UIAlert Alert="this.AlertMessage" SizeCode="Bootstrap.SizeCode.sm"></UIAlert>
        </UIColumn>
        <UIButtonColumn Columns="5">
             .........
        </UIButtonColumn>
    </UIRow>
</UIContainer>
```

##### UIErrorHandler

This component deals with loading operations and errors.  It's inherits directly from UIBase.  It has three states:
1. Loading when it displays the loading message and the spinner.
2. Error when it displays an error message.
3. Loaded when it displays the Child Content.

The state is controlled by the two boolean Parameters.  Content is only accessed and rendered when the control knows there's data to render i.e. when *IsError* and *IsLoading* are both false.  This saves implementing a lot of error checking in the child content.

```c#
// CEC.Blazor/Components/UIControls/UI/UIErrorHandler.cs
public class UIErrorHandler : UIBase
{
    /// Boolean Property that determines if the child content or an error message is diplayed
    [Parameter]
    public bool IsError { get; set; } = false;

    /// Boolean Property that determines if the child content or an loading message is diplayed
    [Parameter]
    public bool IsLoading { get; set; } = true;

    /// CSS Override
    protected override string _BaseCss => this.IsLoading? "text-center p-3": "label label-error m-2";

    /// Customer error message to display
    [Parameter]
    public string ErrorMessage { get; set; } = "An error has occured loading the content";

        
    protected override void BuildRenderTree(RenderTreeBuilder builder)
    {
        this.ClearDuplicateAttributes();
        if (IsLoading)
        {
            builder.OpenElement(1, "div");
            builder.AddAttribute(2, "class", this._Css);
            builder.OpenElement(3, "button");
            builder.AddAttribute(4, "class", "btn btn-primary");
            builder.AddAttribute(5, "type", "button");
            builder.AddAttribute(6, "disabled", "disabled");
            builder.OpenElement(7, "span");
            builder.AddAttribute(8, "class", "spinner-border spinner-border-sm pr-2");
            builder.AddAttribute(9, "role", "status");
            builder.AddAttribute(10, "aria-hidden", "true");
            builder.CloseElement();
            builder.AddContent(11, "  Loading...");
            builder.CloseElement();
            builder.CloseElement();
        }
        else if (IsError)
        {
            builder.OpenElement(1, "div");
            builder.OpenElement(2, "span");
            builder.AddAttribute(3, "class", this._Css);
            builder.AddContent(4, ErrorMessage);
            builder.CloseElement();
            builder.CloseElement();
        }
        else builder.AddContent(1, ChildContent);
    }
}
```

Here's some code showing the control in use.

```html
// CEC.Weather/Components/Forms/WeatherViewer.razor
<UICard>
    <Header>
        @this.PageTitle
    </Header>
    <Body>
        <UIErrorHandler IsError="this.IsError" IsLoading="this.IsDataLoading" ErrorMessage="@this.RecordErrorMessage">
            <UIContainer>
            ..........
            </UIContainer>
        </UIErrorHandler>
        .......
    </Body>
```

##### UIContainer/UIRow/UIColumn

Thess creates a BootStrap Container, Row and Column.  They build out DIVs with the correct Css.

```c#
// CEC.Blazor/Components/UIControls/UIBootstrapContainer/UIContainer.cs
    public class UIContainer : UIBase
    {
        // Overrides the _BaseCss property to force the css_
        protected override string _BaseCss => "container-fluid";
    }
```


```c#
// CEC.Blazor/Components/UIControls/UIBootstrapContainer/UIRow.cs
    public class UIRow : UIBase
    {
        protected override string _BaseCss => "row";
    }
```

```c#
// CEC.Blazor/Components/UIControls/UIBootstrapContainer/UIColumn.cs
public class UIColumn : UIBase
{
    [Parameter]
    public int Columns { get; set; } = 1;

    protected override string _BaseCss => $"col-{Columns}";
}
```

```c#
// CEC.Blazor/Components/UIControls/UIBootstrapContainer/UILabelColumn.cs
public class UILabelColumn : UIColumn
{
    protected override string _BaseCss => $"col-{Columns} col-form-label";
}
```

Here's some code showing the controls in use.

```html
// CEC.Weather/Components/Forms/WeatherViewer.razor
<UIContainer>
    <UIRow>
        <UILabelColumn Columns="2">
            Date
        </UILabelColumn>
        ............
    </UIRow>
..........
</UIContainer>
```

### Wrap Up
This article provides an overview on building UI Controls with components, and examines some example components in more detail.  You can see all the library UIControls in the GitHub Repository - [CEC.Blazor/Components/UIControls](https://github.com/ShaunCurtis/CEC.Blazor/tree/master/CEC.Blazor/Components/UIControls)

Some key points to note:
1. UI Controls lets you abstract the markup from your components.
2. UI Controls gives you project control over the HTML and Css markup.
3. Your main View and Form components are much cleaner and easier to view.
4. You can use as little or as much abstraction as you wish.

