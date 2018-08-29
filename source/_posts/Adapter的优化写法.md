---
path: Android/Adapter的优化写法.md
title: Adapter的优化写法
date: 2016-11-14T22:29:00.000Z
updated: 2017-11-30T14:27:00.000Z
tags:
    - Android
categories:
    - Android
---

ViewHolder不再单纯是View的容器，也承担了View实例化的操作。

```java
public class CustomAdapter extends BaseAdapter {

    private List<String> mDataList;

    public CustomAdapter(List<String> dataList) {
        mDataList = dataList;
    }

    @Override
    public int getCount() {
        return mDataList.size();
    }

    @Override
    public Object getItem(int position) {
        return mDataList.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder = null;

        if (convertView == null){
            convertView = LayoutInflater.from(parent.getContext())
                    .inflate(R.layout.item_custom_list, parent, false);
            viewHolder = new ViewHolder(convertView);
            convertView.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder) convertView.getTag();
        }

        viewHolder.mTv.setText("this is test text");
        return convertView;
    }

    public static class ViewHolder {
        TextView mTv;
        public ViewHolder(View view) {
            mTv = (ImageView) view.findViewById(R.id.text_view);
        }
    }
}
```