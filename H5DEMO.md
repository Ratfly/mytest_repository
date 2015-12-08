### 1、自定义TitleBar示例

```
public class MyTitltBarImpl extends H5TitleBar {
    public MyTitltBarImpl(Context context) {
        super(context);
        contentView.setBackgroundColor(Color.RED);
    }
}
```

### 2、自定义WebView背景色示例

```
public class MyWebContent extends H5WebContentImpl {
    public MyWebContent(Context context) {
        super(context);
        tvProvider.setTextColor(0x60FF0000);//更改DOMAIN颜色
        webContent.setBackgroundColor(0x60FF0000);//更改背景色
    }
}
```

### 2、自定义右上角三点View示例

```
public class MyNavMenu extends H5NavMenu {
    @Override
    public void setList(List<NavMenuItem> list) {
        super.setList(list);
        menuList.add(new NavMenuItem("测试","test",null,false));
    }

    @Override
    public int getListCount() {
        return super.getListCount();
    }

    @Override
    public View getItemView(int position, ViewGroup parent) {
        return super.getItemView(position, parent);
    }
}
```