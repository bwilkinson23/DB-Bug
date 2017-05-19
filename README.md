# DB-Bug
This is documentation on a bug I found while using Tutorials Point Sqlite guide for Android

##Scenario:##
If you were to add 2 'contacts', the 2nd one
would have the ID of 2. 

If you then deleted the first entry, when you go to view the remaining entry, the DB would be 
searched for the row with ID 1. This would cause a NullPointerException.

Furthermore, if you were to add another item (Id: 3), when you went to retreive it from 
the listView in the MainActivity, it would return the information for the entry of ID 2.

This is caused by the following code:
```
      obj = (ListView)findViewById(R.id.listView1);
      obj.setAdapter(arrayAdapter);
      obj.setOnItemClickListener(new OnItemClickListener(){
         @Override
         public void onItemClick(AdapterView<?> arg0, View arg1, int arg2,long arg3) {
            // TODO Auto-generated method stub
            int id_To_Search = arg2 + 1;
            
            Bundle dataBundle = new Bundle();
            dataBundle.putInt("id", id_To_Search);
            
            Intent intent = new Intent(getApplicationContext(),DisplayContact.class);
            
            intent.putExtras(dataBundle);
            startActivity(intent);
         }
      });
   }
```
The important thing here is that the `onItemClick` is using `arg2` which is drawn from
the 'position' of the item clicked in the list. So in the example, after the ID 1 contact
is deleted, the ID2 item is moved up in the list to be in the 0th position in the list.

The solution I used may not be optimal. I am still new to Android and programming, in general, so
I welcome feedback to my solution.

First, I added a method in the DBHelper.java class as follows:

```
    public int getRowId(int i) {
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor res = db.rawQuery("SELECT id from studytracker", null);
        res.moveToFirst();
        for (int c = 0; i>c; c++) {
            res.moveToNext();
        }
        int lastId = res.getInt(0);
        return lastId;
    }
```

Then I used the position requested in the onItemClick as the argument for the getRowId method
as follows:
(note: variable names differ)

```
        mydb = new DBHelper(this);
        obj.setOnItemClickListener(new OnItemClickListener(){
            @Override
            public void onItemClick(AdapterView<?> displayText, View listXML, int listPosition,long listRowID) {
                // TODO Auto-generated method stub

                int id_To_Search = mydb.getRowId(listPosition);

                Bundle dataBundle = new Bundle();
                dataBundle.putInt("id", id_To_Search);

                Intent intent = new Intent(getApplicationContext(),DisplayContact.class);

                intent.putExtras(dataBundle);
                startActivity(intent);
            }
        });
```

If there is some understanding of the original code that I missed, please let me know.

Best wishes!
Ben W
