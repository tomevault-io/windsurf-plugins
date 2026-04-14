---
trigger: always_on
description: Áp dụng các rules này cho **mọi file XAML** trong project (WPF / UWP / MAUI).
---

# XAML Design Rules — GitHub Copilot Instructions

Áp dụng các rules này cho **mọi file XAML** trong project (WPF / UWP / MAUI).

---

## Core Mental Model

```
Chiều DỌC  → StackPanel (hoặc ItemsControl)
Chiều NGANG → Grid với ColumnDefinitions
```

**Không bao giờ** dùng cả `RowDefinitions` lẫn `ColumnDefinitions` trong cùng một Grid nếu có thể tránh được.  
Mỗi Grid chỉ nên giải quyết **một chiều**.

---

## Rule 1 — StackPanel thay thế RowDefinitions

❌ Sai — Grid vừa Row vừa Column:
```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="150"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>
    <TextBlock Text="Name" />
    <TextBlock Grid.Column="1" Text="{Binding Name}" />
    <TextBlock Grid.Row="1" Text="Email" />
    <TextBlock Grid.Row="1" Grid.Column="1" Text="{Binding Email}" />
</Grid>
```

✅ Đúng — StackPanel + Grid thuần cột:
```xml
<StackPanel>
    <Grid Margin="0,0,0,8">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="150"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <TextBlock Text="Name" />
        <TextBlock Grid.Column="1" Text="{Binding Name}" />
    </Grid>
    <Grid Margin="0,0,0,8">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="150"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <TextBlock Text="Email" />
        <TextBlock Grid.Column="1" Text="{Binding Email}" />
    </Grid>
</StackPanel>
```

---

## Rule 2 — Không hard-code Row/Column index

❌ Sai — index rải rác, dễ vỡ khi thêm dòng:
```xml
<TextBlock Grid.Row="2" Grid.Column="1" Text="..." />
<Button    Grid.Row="2" Grid.Column="2" Content="Copy" />
```

✅ Đúng — mỗi Grid là một đơn vị độc lập, không cần index Row.

---

## Rule 3 — Pattern lặp ≥ 3 lần → ItemsControl + DataTemplate

Không copy-paste cùng một cấu trúc Label/Value/Action nhiều lần.  
Dùng `ItemsControl` với `DataTemplate`:

```xml
<ItemsControl ItemsSource="{Binding Fields}" Margin="10">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Grid Margin="0,0,0,8">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="150"/>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <TextBlock Text="{Binding Label}" />
                <TextBlock Grid.Column="1" Text="{Binding Value}" TextWrapping="Wrap" />
                <Button Grid.Column="2" Content="Copy"
                        Command="{Binding CopyCommand}"
                        CommandParameter="{Binding Value}"
                        Padding="10,2" Margin="8,0,0,0"/>
            </Grid>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```

---

## Rule 4 — SharedSizeGroup để căn cột đồng nhau

```xml
<StackPanel Grid.IsSharedSizeScope="True">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition SharedSizeGroup="Label"/>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition SharedSizeGroup="Action"/>
        </Grid.ColumnDefinitions>
        ...
    </Grid>
    <!-- Grid khác cùng group tự căn theo -->
</StackPanel>
```

`Grid.IsSharedSizeScope="True"` đặt trên **phần tử cha**, không phải từng Grid con.

---

## Rule 5 — UserControl cho pattern phức tạp

Khi một row có logic (tooltip, validation, visibility binding) và dùng ≥ 2 nơi:

```xml
<!-- FieldRow.xaml -->
<UserControl x:Class="App.Controls.FieldRow"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Name="root">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="150"/>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="Auto"/>
        </Grid.ColumnDefinitions>
        <TextBlock Text="{Binding Label, ElementName=root}" VerticalAlignment="Center"/>
        <TextBlock Grid.Column="1" Text="{Binding Value, ElementName=root}"
                   TextWrapping="Wrap" VerticalAlignment="Center"/>
        <Button Grid.Column="2" Content="Copy"
                Command="{Binding CopyCommand, ElementName=root}"
                CommandParameter="{Binding Value, ElementName=root}"
                Padding="10,2" Margin="8,0,0,0"/>
    </Grid>
</UserControl>
```

---

## Rule 6 — Thứ tự ưu tiên layout

```
1. StackPanel    → xếp chồng đơn giản, không cần căn cột
2. Grid (column) → căn nhiều cột trong một dòng
3. Grid (2D)     → lưới thực sự: calendar, dashboard tile, table
4. UniformGrid   → lưới đều, không cần binding
5. WrapPanel     → tag cloud, chip list
6. DockPanel     → shell layout (header / footer / sidebar)
```

---

## Rule 7 — Spacing tập trung ở container

❌ Sai — margin rải rác từng element:
```xml
<TextBlock Margin="0,0,8,8" />
<TextBlock Margin="0,0,0,8" />
<Button Margin="8,0,0,8" />
```

✅ Đúng — gom vào `ItemContainerStyle`:
```xml
<ItemsControl>
    <ItemsControl.ItemContainerStyle>
        <Style TargetType="ContentPresenter">
            <Setter Property="Margin" Value="0,0,0,8"/>
        </Style>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thangworks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
