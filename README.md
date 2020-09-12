# Android-Studio-code-of-PM2.5-Sensor-and-App-Project

![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a00.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a01.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a02.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a03.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a04.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a05.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a06.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a07.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a08.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a09.jpg)
![image](https://github.com/BookCatCSIE/Android-Studio-code-of-PM2.5-Sensor-and-App-Project/blob/master/%E8%A8%88%E6%A6%82project%20%E5%9C%96/a10.jpg)

package com.example.bookcat.textapp;

import android.bluetooth.BluetoothAdapter;
import android.bluetooth.BluetoothDevice;
import android.bluetooth.BluetoothSocket;
import android.content.Intent;
import android.content.res.Resources;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.location.Criteria;
import android.location.LocationManager;
import android.os.Handler;
import android.support.annotation.Nullable;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.text.SimpleDateFormat;
import java.util.Set;
import java.util.UUID;

public class MainActivity extends AppCompatActivity   {
    private Button button_paired;
    private Button button_disconnect;
    private Button button_find;
    private TextView show_data;
    private TextView show_count;
    private ListView event_listview;
    //bluetooth
    private BluetoothAdapter mBluetoothAdapter;
    private ArrayAdapter<String> devicename;
    private ArrayAdapter<String> deviceID;
    private Set<BluetoothDevice> pairedDevices;
    private String choseID;
    private BluetoothDevice bledevice;
    private BluetoothSocket bluesocket;
    private InputStream mmInputStream;
    private OutputStream mmOutputStream;
    Thread workerThread;
    volatile boolean stopWorker;
    private int readBufferPosition;
    private byte[] readBuffer;
    private String uid;
    private int count;
    private int PMvalue;
    private LocationManager locMgr;
    private String bestProv;
    Resources res;
    Bitmap a;
    private TextView DangerText;
    private ImageView image;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        res = getResources();
        a = BitmapFactory.decodeResource(res, R.drawable.a00);
        getview();
        setListener();
        devicename = new ArrayAdapter<String>(this, android.R.layout.simple_expandable_list_item_1);
        deviceID = new ArrayAdapter<String>(this, android.R.layout.simple_expandable_list_item_1);
        count=0;
        image.setImageResource(R.drawable.a00);

    }
    private void getview() {
        button_paired = (Button) findViewById(R.id.btn_paired);
        button_disconnect = (Button) findViewById(R.id.btn_disconn);
        show_data = (TextView) findViewById(R.id.txtShow);
        event_listview = (ListView) findViewById(R.id.Show_B_List);
        button_find = (Button) findViewById(R.id.btn_conn);
        show_count = (TextView) findViewById(R.id.txt_count);
        DangerText = (TextView) findViewById(R.id.DangerText);
        image = (ImageView) findViewById(R.id.pmimage);

    }
    private void setListener() {
        button_paired.setOnClickListener(new Button.OnClickListener(){
            @Override
            public void onClick(View v)
            {
                findPBT();
            }
        });
        button_disconnect.setOnClickListener((v)->{
            try{
                closeBT();
            }catch(IOException e){
                e.printStackTrace();
            }
        });
        button_find.setOnClickListener(new Button.OnClickListener(){
            @Override
            public void onClick(View v) {
                findPBT();
            }
        });
        event_listview.setAdapter(devicename);
        event_listview.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> partent, View view, int position, long id) {
                choseID = deviceID.getItem(position);
                try {
                    openBT();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                Toast.makeText(MainActivity.this, "選擇了:" + choseID, Toast.LENGTH_SHORT).show();
                devicename.clear();
            }
        });
    }
    private void findPBT(){
        mBluetoothAdapter=BluetoothAdapter.getDefaultAdapter();
        if(mBluetoothAdapter==null){
            show_data.setText("No bluetooth adapter available");
        }
        if(!mBluetoothAdapter.isEnabled()){
            Intent enaBlebluetooth=new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
            startActivityForResult(enaBlebluetooth,1);
        }
        pairedDevices=mBluetoothAdapter.getBondedDevices();
        if (pairedDevices.size()>0){
            for(BluetoothDevice device:pairedDevices){
                String str="已配對完成的裝置有:"+device.getName()+" "+device.getAddress()+"\n";
                uid=device.getAddress();
                bledevice=device;
                devicename.add(str);
                deviceID.add(uid);
            }
            event_listview.setAdapter(devicename);
        }
    }
    private void openBT() throws IOException{
        UUID uuid=UUID.fromString("00001101-0000-1000-8000-00805F9B34FB");
        if(bledevice!=null){
            bluesocket=bledevice.createRfcommSocketToServiceRecord(uuid);

            bluesocket.connect();
            mmOutputStream=bluesocket.getOutputStream();
            mmInputStream=bluesocket.getInputStream();

            beginListenForData();

            show_data.setText("Bluetooth Opened: "+bledevice.getName()+" "+bledevice.getAddress());
            View b1=findViewById(R.id.btn_conn);
            View b2=findViewById(R.id.btn_paired);
            View b3=findViewById(R.id.Show_B_List);
            b1.setVisibility(View.INVISIBLE);
            b2.setVisibility(View.INVISIBLE);
            b3.setVisibility(View.INVISIBLE);
        }
    }
    private void beginListenForData(){
        final Handler handler=new Handler();
        final byte delimiter=10;

        stopWorker = false;
        readBufferPosition = 0;
        readBuffer = new byte[1024];
        workerThread = new Thread((Runnable) ()-> {
                while (!Thread.currentThread().isInterrupted() && !stopWorker)
                {
                    try {
                        int bytesavailable = mmInputStream.available();
                        if (bytesavailable > 0)
                        {
                            byte[] packetBytes = new byte[bytesavailable];
                            mmInputStream.read(packetBytes);
                            for (int i = 0; i < bytesavailable; i++) {
                                byte b = packetBytes[i];
                                if (b == delimiter)
                                {
                                    byte[] encodedBytes = new byte[readBufferPosition];
                                    System.arraycopy(readBuffer,0, encodedBytes, 0, encodedBytes.length);
                                    final String data = new String(encodedBytes,"US-ASCII");
                                    PMvalue = 0;
                                    for (int j = 0; j < encodedBytes.length - 1; j++) {
                                        PMvalue = PMvalue * 10 + encodedBytes[j] - 48;
                                    }
                                    String tmp1=String.valueOf(PMvalue);
                                    Log.d("value",tmp1);
                                    readBufferPosition = 0;
                                    count++;
                                    handler.post(()-> {
                                            long date=System.currentTimeMillis();
                                            TextView tvdisplaydate=(TextView)findViewById(R.id.date);
                                            SimpleDateFormat sdf=new SimpleDateFormat("MMM MM dd,yyyy h:mm a");
                                            tvdisplaydate.setText("update time:"+sdf.format(date));
                                            String prevString = show_data.getText().toString();
                                            String dataText = String.format("PM2.5=%sug/m3,收到了第%s筆資料\n%s", data,count,prevString);
                                            show_data.setText(dataText);
                                                if (PMvalue < 36) {
                                                    if (PMvalue > 23) {
                                                        a = BitmapFactory.decodeResource(res, R.drawable.a03);
                                                        image.setImageBitmap(a);
                                                        DangerText.setText("良好");
                                                        show_count.setText(data);
                                                    } else if (PMvalue > 11) {
                                                        a = BitmapFactory.decodeResource(res, R.drawable.a02);
                                                        image.setImageBitmap(a);
                                                        DangerText.setText("良好");
                                                        show_count.setText(data);
                                                    } else if (PMvalue > 0) {
                                                        a = BitmapFactory.decodeResource(res, R.drawable.a01);
                                                        image.setImageBitmap(a);
                                                        DangerText.setText("良好");
                                                        show_count.setText(data);
                                                    }
                                                } else if (PMvalue < 54) {
                                                    if (PMvalue > 47) {
                                                        a = BitmapFactory.decodeResource(res, R.drawable.a06);
                                                        image.setImageBitmap(a);
                                                        DangerText.setText("警戒");
                                                        show_count.setText(data);
                                                    } else if (PMvalue > 41) {
                                                        a = BitmapFactory.decodeResource(res, R.drawable.a05);
                                                        image.setImageBitmap(a);
                                                        DangerText.setText("警戒");
                                                        show_count.setText(data);
                                                    } else if (PMvalue > 35) {
                                                        a = BitmapFactory.decodeResource(res, R.drawable.a04);
                                                        image.setImageBitmap(a);
                                                        DangerText.setText("警戒");
                                                        show_count.setText(data);
                                                    }
                                                } else if (PMvalue < 71) {
                                                    if (PMvalue > 64) {
                                                        a = BitmapFactory.decodeResource(res, R.drawable.a09);
                                                        image.setImageBitmap(a);
                                                        DangerText.setText("過量");
                                                        show_count.setText(data);
                                                    } else if (PMvalue > 58) {
                                                        a = BitmapFactory.decodeResource(res, R.drawable.a08);
                                                        image.setImageBitmap(a);
                                                        DangerText.setText("過量");
                                                        show_count.setText(data);
                                                    } else if (PMvalue > 53) {
                                                        a = BitmapFactory.decodeResource(res, R.drawable.a07);
                                                        image.setImageBitmap(a);
                                                        DangerText.setText("過量");
                                                        show_count.setText(data);
                                                    }
                                                } else if (PMvalue >= 71) {
                                                    a = BitmapFactory.decodeResource(res, R.drawable.a10);
                                                    image.setImageBitmap(a);
                                                    DangerText.setText("危險");
                                                    show_count.setText(data);
                                                }
                                    });
                                } else {
                                    readBuffer[readBufferPosition++] = b;
                                }
                            }
                        }
                    } catch (IOException ex) {
                        stopWorker = true;
                    }
                }
        });
        workerThread.start();
    }
    private void closeBT() throws IOException{
        stopWorker = true;
        mmOutputStream.close();
        mmInputStream.close();
        bluesocket.close();
        devicename.clear();
        show_count.setText("");
        show_data.setText("Bluetooth Closed");
        DangerText.setText("");
    }
}

