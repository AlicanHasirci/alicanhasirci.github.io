---
layout: post
title: "Android, Facebook And Sharing"
date: 2015-10-07 18:05:28
image: '/assets/img/'
description:
tags:
- android
- facebook
- screenshot
- share
categories:
- Android
twitter_text:
---

Lately, i was tasked to integrate 'Screenshot Sharing' on one of ours, [Spades Plus](https://play.google.com/store/apps/details?id=net.peakgames.mobile.spades.android). To simply put it, the task was to allow users share a screenshot with a predefined message for promotion. 

Ideally doing this should be easy where you put your message with your screenshot uri to intent and applications can handle it properly, except for Facebook. Facebook for some reason still unknown to me, does not let you share a link with an image(at least it wasn't when had thet task, may have changed now). At first i thought this was from a was a design policy, then asking the iOS developer tasked with the same job made me believe otherwise. On iOS you can share a link with an image but on Android you cannot. Doing both one at a time was not an option thanks to product manager so i had to come up with a new game plan. 

Thought proccess went like this. I could share both(link and picture) on facebook as a story but this would require a different action path to take from other social media channels and i didn't want to give up on them so easily, so there had to be a choice for user to pick from. I could not do it with android's own intent chooser, since i cannot get a callback from i put my hacker hat on. After considering bunch of ideas i settled with developing my own IntentChooser which would look and feel like the native one, but take action according to users choice.

So here is what the class looks like:


	public class ChooserDialog extends Dialog{

	    public ChooserDialog(Context context, String title,Intent intent, final Callback callback, String... filter) {
	        super(context);
	        LinearLayout dialogLayout = new LinearLayout(context);

	        LinearLayout.LayoutParams dialogLayoutParams = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT,LinearLayout.LayoutParams.MATCH_PARENT);
	        dialogLayout.setLayoutParams(dialogLayoutParams);
	        dialogLayout.setOrientation(LinearLayout.VERTICAL);

	        this.setTitle(title);

	        List<ResolveInfo> infoList = context.getPackageManager().queryIntentActivities(intent, 0);
	        List<ResolveInfo> selectedInfoList = new ArrayList<ResolveInfo>();
	        if(filter.length > 0){
	            for(ResolveInfo info : infoList){
	                if(existsInFilter(info, filter)){
	                    selectedInfoList.add(info);
	                }
	            }
	        }else{
	            selectedInfoList = infoList;
	        }
	        DialogListAdapter adapter = new DialogListAdapter(context, selectedInfoList, new Callback() {
	            @Override
	            public void itemClicked(String packageName) {
	                callback.itemClicked(packageName);
	                ChooserDialog.this.dismiss();
	            }
	        });

	        ListView list = new ListView(context);
	        list.setLayoutParams(new ViewGroup.LayoutParams(AbsListView.LayoutParams.WRAP_CONTENT, AbsListView.LayoutParams.MATCH_PARENT));
	        list.setAdapter(adapter);

	        dialogLayout.addView(list);

	        this.setContentView(dialogLayout);
	    }

	    private boolean existsInFilter(ResolveInfo info, String... filter){
	        for(int i=0;i<filter.length;i++){
	            if(info.activityInfo.packageName.equals(filter[i])){
	                return true;
	            }
	        }
	        return false;
	    }

	    public interface Callback{
	        public void itemClicked(String packageName);
	    }
	}

One thing to notice here is that i've implemented a filter to filter out unwanted packages. Also there are some missing classes, DialogListAdapter and DialogListItem that the adapter uses. Their job is pretty much obvious but i'll elaborate. Basicly adapter only gets the ResolveInfo list to pass it during the creation of DialogListItem then calls the callback given on items click.

	public class DialogListAdapter extends BaseAdapter {

	    private Context context;
	    private List<ResolveInfo> infoList = Collections.emptyList();
	    private ChooserDialog.Callback callback;

	    DialogListAdapter(Context context, List<ResolveInfo> infoList, ChooserDialog.Callback callback){
	        this.infoList = infoList;
	        this.context = context;
	        this.callback = callback;
	    }

	    @Override
	    public int getCount() {
	        return infoList.size();
	    }

	    @Override
	    public Object getItem(int i) {
	        return infoList.get(i);
	    }

	    @Override
	    public long getItemId(int i) {
	        return 0;
	    }

	    @Override
	    public View getView(final int i, View view, ViewGroup viewGroup) {
	        DialogListItem item = new DialogListItem(context, infoList.get(i));
	        item.setOnClickListener(new View.OnClickListener() {
	            @Override 
	            public void onClick(View view) {
	                callback.itemClicked(infoList.get(i).activityInfo.packageName);
	            }
	        });

	        return item;
	    }
	}


And here is the DialogListItem class that represents every item on the list.


	// I urge you to create your layout as an XML, i could not due to some project-related restrictions.
	public class DialogListItem extends LinearLayout {

	    private static final int ICON_SIZE = 40;
	    private static final int TEXT_SIZE = 20;
	    private static final int ITEM_HEIGHT = 50;
	    private static final int TEXT_PADDING_LEFT = 5;
	    private static final int LIST_ITEM_PADDING = 5;

	    private ImageView icon;
	    private TextView text;

	    public DialogListItem(Context context, ResolveInfo resolveInfo) {
	        super(context);
	        this.setOrientation(LinearLayout.HORIZONTAL);
	        this.setLayoutParams(new AbsListView.LayoutParams(AbsListView.LayoutParams.WRAP_CONTENT, convert(ITEM_HEIGHT)));
	        this.setGravity(Gravity.CENTER_VERTICAL);
	        int convertedPadding = convert(LIST_ITEM_PADDING);
	        this.setPadding(convertedPadding,
	                convertedPadding,
	                convertedPadding,
	                convertedPadding);

	        icon = new ImageView(context);
	        icon.setImageDrawable(resize(resolveInfo.loadIcon(context.getPackageManager())));
	        this.addView(icon);

	        text = new TextView(context);
	        text.setText(resolveInfo.loadLabel(context.getPackageManager()));
	        text.setTextSize(TEXT_SIZE);
	        text.setPadding(convert(TEXT_PADDING_LEFT), 0, 0, 0);
	        this.addView(text);
	    }

	    private int convert(float dp){
	        final float scale = getContext().getResources().getDisplayMetrics().density;
	        return (int) (dp * scale);
	    }

	    private Drawable resize(Drawable image) {
	        Bitmap b = ((BitmapDrawable)image).getBitmap();
	        int size = convert(ICON_SIZE);
	        Bitmap bitmapResized = Bitmap.createScaledBitmap(b, size, size, false);
	        return new BitmapDrawable(getResources(), bitmapResized);
	    }
	}


From this point on, say you want to share an image with different behaviour for every social media:


	Intent shareIntent = new Intent(Intent.ACTION_SEND);
    shareIntent.setType("image/png");
    ChooserDialog.Callback callback = new ChooserDialog.Callback() {
        @Override
        public void itemClicked(String packageName) {
            // Choice specific behaviour
        }
    };
    ChooserDialog dialog = new ChooserDialog(activity, chooserTitle, shareIntent, callback, filter);
    dialog.show();


What i did after this was to handle the callback so that if it is facebook:

* Upload screenshot to one of our servers the then get the URL from response.
* Publish the message with URL url as a story to users wall.

If not, create an intent and send it to the corresponding package.

