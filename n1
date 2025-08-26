// MainActivity.java
package com.example.plchmi;

import android.os.Bundle;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

import Moka7.S7;
import Moka7.S7Client;

public class MainActivity extends AppCompatActivity {

    private S7Client client = new S7Client();
    private final String plcIp = "192.168.0.1"; // <-- replace with your PLC IP
    private final int rack = 0; // usually 0 for S7-1200
    private final int slot = 1; // usually 1 for S7-1200

    private TextView valueText;
    private Button startButton, stopButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        valueText = findViewById(R.id.valueText);
        startButton = findViewById(R.id.startButton);
        stopButton = findViewById(R.id.stopButton);

        // Connect to PLC in a background thread
        new Thread(() -> {
            int res = client.ConnectTo(plcIp, rack, slot);
            runOnUiThread(() -> {
                if (res == 0) {
                    Toast.makeText(this, "Connected to PLC", Toast.LENGTH_SHORT).show();
                    readPlcValue();
                } else {
                    Toast.makeText(this, "Connection failed: " + client.ErrorText(res), Toast.LENGTH_LONG).show();
                }
            });
        }).start();

        startButton.setOnClickListener(v -> writeMotorBit(true));
        stopButton.setOnClickListener(v -> writeMotorBit(false));
    }

    private void readPlcValue() {
        new Thread(() -> {
            try {
                byte[] buffer = new byte[4];
                int result = client.DBRead(1, 0, 4, buffer); // DB1.DBD0
                if (result == 0) {
                    float value = S7.GetFloatAt(buffer, 0);
                    runOnUiThread(() -> valueText.setText("Analog: " + value));
                }
            } catch (Exception e) {
                runOnUiThread(() -> Toast.makeText(this, "Read error: " + e.getMessage(), Toast.LENGTH_LONG).show());
            }
        }).start();
    }

    private void writeMotorBit(boolean start) {
        new Thread(() -> {
            try {
                byte[] buffer = new byte[1];
                // Read current byte, so we don't overwrite other bits in DB1.DBX0
                client.DBRead(1, 0, 1, buffer);
                S7.SetBitAt(buffer, 0, 0, start); // set bit 0.0 true/false
                int result = client.DBWrite(1, 0, 1, buffer);
                runOnUiThread(() -> {
                    if (result == 0) {
                        Toast.makeText(this, start ? "Motor Started" : "Motor Stopped", Toast.LENGTH_SHORT).show();
                    } else {
                        Toast.makeText(this, "Write failed: " + client.ErrorText(result), Toast.LENGTH_LONG).show();
                    }
                });
            } catch (Exception e) {
                runOnUiThread(() -> Toast.makeText(this, "Write error: " + e.getMessage(), Toast.LENGTH_LONG).show());
            }
        }).start();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        client.Disconnect();
    }
}
