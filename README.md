# sc-max3010x

SuperCollider plugin to read data from MAX30102 and MAX30105 over i2c.

Based on https://github.com/jpburstrom/sc-mpu9250

This project will build:

 * MPU: a SuperCollider UGen plugin
 * debug: an executable that reads the data from MAX3010x over i2c and prints it

### To build:

    mkdir build
    cd build
    cmake -DSC_PATH=/root/supercollider -DCMAKE_C_FLAGS="-march=armv7-a -mtune=cortex-a8 -mfloat-abi=hard -mfpu=neon -O2" -DCMAKE_CPP_FLAGS="-march=armv7-a -mtune=cortex-a8 -mfloat-abi=hard -mfpu=neon -O2" ..
    make 

#### To get supercollider and copy it to Bela

    git clone --recurse-submodules https://github.com/SuperCollider/SuperCollider.git
    scp -r "SuperCollider" "root@bela.local:/root/supercollider/"

### To test / debug:

    ./build/apps/MAX30105_debug/MAX30105_debug
    
### To install SuperCollider extension

    cp plugin/MAX30105.so ~/.local/share/SuperCollider/Extensions/MAX30105/plugins/
    cp plugin/MAX30105.sc ~/.local/share/SuperCollider/Extensions/MAX30105/classes/
    
### In SuperCollider
    Example 1:
    ```
    { Max30102.kr(0) * WhiteNoise.ar!2 * 0.1 }.play; // white noise modulated by pulse
    ```

    Example 2:
    ```
    s = Server.default;

    s.options.numAnalogInChannels = 8; // can be 2, 4 or 8
    s.options.numAnalogOutChannels = 8; // can be 2, 4 or 8
    s.options.numDigitalChannels = 16;
    s.options.maxLogins = 8;
    // s.options.bindAddress = "0.0.0.0"; // allow anyone on the network connect to this server

    s.options.pgaGainLeft = 5;     // sets the pregain for the left audio input (dB)
    s.options.pgaGainRight = 5;    // sets the pregain for the right audio input (dB)
    s.options.headphoneLevel = 0; //-1; // sets the headphone level (-dB)
    s.options.speakerMuted = 0;    // set true to mute the speaker amp and draw a little less power
    s.options.dacLevel = 0;       // sets the gain of the stereo audio dac (+dB)
    s.options.adcLevel = 0;       // sets the gain of the stereo audio adc (+dB)

    s.options.blockSize = 128; //16;
    s.options.numInputBusChannels = 2; //10;
    s.options.numOutputBusChannels = 2;


    s.waitForBoot {
      SynthDef(\max30102, {
        // channels
        // 0: IR_AC
        // 1: IR_BEAT
        // 2: IR_RAW
        // 3: RED_RAW
        // 4: IR_BPM
        // 5: HZ
        // 6: TEMP_C
        // 7: IR_AVG_DC_EST

        // var ir_ac = Max30102.kr(0);
        // var ir_beat = Max30102.kr(1);
        // var ir_raw = Max30102.kr(2);
        // var red_raw = Max30102.kr(3);
        var ir_bpm = Max30102.kr(4);
        // var hz = Max30102.kr(5);
        // var temp_c = Max30102.kr(6);
        // var ir_avg_dc_est = Max30102.kr(7);

        SendReply.kr(Impulse.kr(10), "/max30102", [
          ir_bpm
        ]);
      }).send(s);

      s.sync;

      ~max30102 = Synth(\max30102);

      OSCdef(\max30102, {|msg| 
        msg.postln;
      }, "/max30102");
    }
    ```
