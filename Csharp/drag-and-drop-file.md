# Drag and Drop a File onto a Windows Forms Application

```cs
InitializeComponent();
AllowDrop = true;
DragEnter += DragEnter;
DragDrop += DragDrop;
```

```cs
private void DragEnter(object sender, DragEventArgs e)
{
    if (e.Data.GetDataPresent(DataFormats.FileDrop))
        e.Effect = DragDropEffects.Copy;
}
```

```cs
private void DragDrop(object sender, DragEventArgs e)
{
    string[] paths = (string[])e.Data.GetData(DataFormats.FileDrop);
    foreach (string path in paths)
    {
        Console.WriteLine(path);
    }
}
```
