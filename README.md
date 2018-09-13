# VirtualizationListView

*VirtualizationListView* is WPF *ListView* *UserControl* element, designed for high performance large data sets representation. This purpose is reached by applying UI and data virtualization technics. The **key features** of *VirtualizationListView* are:
* **Mutable data** support with lightweight data changes handling
* Fast **sorting**
* Flexible **filtering** by grid columns and by complex filter conditions
* Presentation of **objects of different classes in one grid**
* **Localization** capability

The library is built on the .NET Framework 4.5 in the Visual Studio 2017 and based on MVVM pattern. It is completely **free** and **open source**. 

More about implemented data virtualization algorithm and other details you can learn from our paper: [Displaying large sets of dynamically changing remote data in WPF applications](http://injoit.org/index.php/j1/article/view/485)

# How to use

**An example of use you can find in the source code**

To use library from your project:
* Add reference to library from your project
* Add **dependencies** to the library:
  * System.Threading.Tasks.Dataflow
  * System.Windows.Interactivity
* Define classes of objects you want to present in *VirtualizationListView* (objects in a list view can be of different classes). In our example we suggest you have *BaseDataType* as superclass with property *CommonProperty* and its child classes *ChildDataType1* and *ChildDataType2* with properties *Property1* and *Property2* accordingly.

## Implementing *IItemsProvider\<T>* interface to provide data for *VirtualizationListView*

* Implement *IItemsProvider\<T>* interface to fetch data from your data source (on request from the database or using DynamicLinq or others)

```csharp

```

Data virtualization is provided by *VirtualizingCollection\<T>* class.

## Describing columns for *VirtualizationListView* grid

* Add new resource file to your project (named, for example, "YourListView.xaml")
* Create new *ResourceDictionary* inside "YourListView.xaml":
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:slaveTypes="clr-namespace:VirtualizationListViewControl.SlaveTypes;assembly=VirtualizationListViewControl"
  xmlns:helpers="clr-namespace:VirtualizationListViewControl.Helpers;assembly=VirtualizationListViewControl"
  xmlns:sortAndFilterDto="clr-namespace:VirtualizationListView.SortAndFilterDTO;assembly=VirtualizationListView.SortAndFilterDTO"
  xmlns:system="clr-namespace:System;assembly=mscorlib"
  xmlns:yourAppSpecificDataTypes="clr-namespace:YourAppSpecificDataTypes;assembly=YourAppSpecificDataTypesAssembly"
  xmlns:glob="clr-namespace:System.Globalization;assembly=mscorlib">

  <slaveTypes:VirtualizationListViewColumnCollection x:Key="YourListViewStyle">
    <!-- Column definition with common DataTemplate for all data types -->
    <slaveTypes:VirtualizationListViewColumn Header="Common property column header"
                                             Width="100"
                                             DisabledSorting="True">
      <slaveTypes:VirtualizationListViewColumn.CellTemplate>
        <DataTemplate>  
          <!-- You can describe any template here -->
          <TextBox Text="{Binding Data.CommonProperty, Mode=OneWay}"/>
        </DataTemplate>
      </slaveTypes:VirtualizationListViewColumn.CellTemplate>
      <slaveTypes:VirtualizationListViewColumn.BoundProperty>
        <!-- Describe your binded property here -->
        <sortAndFilterDto:FieldDescription FieldName="CommonProperty" 
                                           Assembly="YourAppSpecificDataTypesAssembly" 
                                           DeclaringType="YourAppSpecificDataTypesAssembly.BaseDataType"/>
      </slaveTypes:VirtualizationListViewColumn.BoundProperty>
    </slaveTypes:VirtualizationListViewColumn>
    
    <!-- Column definition with different DataTemplates for all data types -->
    <slaveTypes:VirtualizationListViewColumn Header="Property1 column header"
                                             Width="100"
                                             DisabledSorting="True">
      <slaveTypes:VirtualizationListViewColumn.CellTemplate>
        <DataTemplate>
          <ContentControl Content="{Binding Data, Mode=OneWay}"
                          Margin="10,3">
            <ContentControl.Resources>
            
              <DataTemplate DataType="{x:Type yourAppSpecificDataTypes:ChildDataType1}">  
                <!-- You can describe any template here -->
                <TextBox Text="{Binding Data.Property1, Mode=OneWay}"/>
              </DataTemplate>

              <!-- If you want no value in current column for ChildDataType2, you should define empty DataTemplate -->
              <DataTemplate DataType="{x:Type yourAppSpecificDataTypes:ChildDataType2}"/>
              
            </ContentControl.Resources>
          </ContentControl>
        </DataTemplate>
      </slaveTypes:VirtualizationListViewColumn.CellTemplate>
      <slaveTypes:VirtualizationListViewColumn.BoundProperty>
        <!-- Describe your binded property here -->
        <!-- Set only one property for sorting -->
        <sortAndFilterDto:FieldDescription FieldName="Property1" 
                                           Assembly="YourAppSpecificDataTypesAssembly" 
                                           DeclaringType="YourAppSpecificDataTypesAssembly.BaseDataType"/>
      </slaveTypes:VirtualizationListViewColumn.BoundProperty>      
    </slaveTypes:VirtualizationListViewColumn>
  </slaveTypes:VirtualizationListViewColumnCollection>    
</ResourceDictionary>
```

## [Optional] Configure the necessary filters for your data



## Adding the *VirtualizationListView* to your XAML code

* Define the *VirtualizationListView* namespace and localization in your *\<Window>* tag:
```xml
<Window x:Class="YourWpfApplication.Views.AppWindow"
  xmlns:virtualizationListViewControl="clr-namespace:VirtualizationListViewControl.Controls;assembly=VirtualizationListViewControl"
  xmlns:localization="clr-namespace:YourWpfApplication.Localization">  
  <Window.Resources>
    <localization:VirtualizationListLocalization x:Key="VirtualizationListLocalization"/>
  </Window.Resources>
  ...
</Window>
```

* Place the *VirtualizationListView* somewhere in your *\<Window>* tag:
```xml
<virtualizationListViewControl:VirtualizationListView x:Name="YourListView" 
  Localization="{Binding ResourceManager, Source={StaticResource VirtualizationListLocalization}}"
  Columns="{StaticResource YourListViewStyle}"
  ItemsSource="{Binding YourListViewDataSource, IsAsync=True}"
  SelectedItem="{Binding YourListViewSelectedItem}"/>
```
Notice, that *Columns* property contains *YourListViewStyle* we created earlier.
