# Hello-world




package cn.itcast.myapplication;

import androidx.annotation.NonNull;
import androidx.annotation.RequiresApi;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.database.Cursor;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.provider.ContactsContract;
import android.util.Log;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.ListView;

import java.util.ArrayList;
import java.util.List;


@RequiresApi(api = Build.VERSION_CODES.M)
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "UserInfo";
    ListView contactsView;
    ArrayAdapter<String> adapter;
    List<String> contactsList = new ArrayList<>();
    public static final int PERMISSION_REQUEST_CODE = 1;

    private Uri contactsUri = ContactsContract.Contacts.CONTENT_URI;// 联系人的Uri对象
    private Uri phoneUri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI;// 获取联系人电话的Uri
    private Uri emailUri = ContactsContract.CommonDataKinds.Email.CONTENT_URI;// 获取联系人邮箱的Uri

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // getUserInfo();
        checkContactReadPermission();
    }

    private void checkContactReadPermission() {
        int readContactPermission = checkSelfPermission(Manifest.permission.READ_CONTACTS);
        if (readContactPermission != PackageManager.PERMISSION_GRANTED) {
            // 需要请求权限
            requestPermissions(new String[] {Manifest.permission.READ_CONTACTS}, PERMISSION_REQUEST_CODE);
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if(requestCode == PERMISSION_REQUEST_CODE) {
            //判断结果
            if(grantResults.length == 1 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                Log.d(TAG,"has permissions..");
                //有权限
            } else {
                Log.d(TAG,"no permissions...");
            }
        }
    }
    public void getContactInfo(View view) {

            if(ContextCompat.checkSelfPermission(MainActivity.this,
                    Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED)
            {
                ActivityCompat.requestPermissions(MainActivity.this,
                        new String[]{Manifest.permission.READ_CONTACTS},1);
            }
            else {
                getUserInfo();
            }


        //getUserInfo();
    }

    private void getUserInfo() {
        contactsList.clear();
        contactsView = findViewById(R.id.contacts_view);
        adapter = new ArrayAdapter<>(this, android.R.layout.simple_expandable_list_item_1, contactsList);
        contactsView.setAdapter(adapter);
        Cursor cursor;
        cursor=getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
                null,null,null,null);
        assert cursor != null;
        while (cursor.moveToNext()) {
            StringBuilder msg= new StringBuilder();
            int contactId = cursor.getInt(cursor
                    .getColumnIndex(ContactsContract.Contacts.NAME_RAW_CONTACT_ID));
            // 获取联系人的name
            String contactName = cursor.getString(cursor
                    .getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME));
            msg.append("姓名：").append(contactName);
            // 通过联系人的id 获取联系人电话信息
            Cursor phoneCursor=getContentResolver().query(phoneUri, null,
                    ContactsContract.CommonDataKinds.Phone.RAW_CONTACT_ID+"=?",
                    new String[]{contactId+""}, null);
            assert phoneCursor != null;
            while (phoneCursor.moveToNext()) {
                // 获取联系人电话号码
                String phone = phoneCursor
                        .getString(phoneCursor
                                .getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));

                msg.append("\n电话号码：").append(phone);
            }

            Cursor emailCursor = getContentResolver().query(emailUri, null,
                    ContactsContract.CommonDataKinds.Email.RAW_CONTACT_ID
                            + "=?", new String[] { contactId + "" }, null);
            assert emailCursor != null;
            while (emailCursor.moveToNext()) {
                // 获取联系人邮箱
                String email = emailCursor
                        .getString(emailCursor
                                .getColumnIndex(ContactsContract.CommonDataKinds.Email.DATA));
                msg.append("\n邮箱：").append(email);
            }
            contactsList.add(msg.toString());
//            Log.d("MainActivity", msg.toString());
        }
        ArrayAdapter<String> adapter1 = new ArrayAdapter<>(this, android.R.layout.simple_expandable_list_item_1, contactsList);
        contactsView.setAdapter(adapter1);
    }
}
