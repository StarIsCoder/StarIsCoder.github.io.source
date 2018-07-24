#Modify shape attributes in code
```
((GradientDrawable)(ContextCompat.getDrawable(context, R.drawable.pen_color_red)))
    .setSize(getResources().getDimensionPixelSize(R.dimen.wb_native_buttons_size_newui),
            getResources().getDimensionPixelSize(R.dimen.wb_native_buttons_size));
```
convert drawable to GradientDrawable,and invoke setSize or setStroke and so on.