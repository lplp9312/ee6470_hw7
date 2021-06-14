# HW7 Report 蕭詠倫 109061634

### 結果

#### 單核心
![image](https://github.com/lplp9312/ee6470_hw6/blob/master/gau_sim.jpg)

#### 雙核心
![image](https://github.com/lplp9312/ee6470_hw7/blob/master/core0_sim.jpg)
![image](https://github.com/lplp9312/ee6470_hw7/blob/master/core1_sim.jpg)

### Testbench code : 把圖片分成左右2個部份，分給filter0和filter1處理，進行RGB傳送和接收 (src/gauss_multi_core/main.cpp)
    
    int width_start = (hart_id == 0) ? 0 : width/2;
    int width_end = (hart_id == 0) ? width/2 : width;
    
      for(int i = width_start; i < width_end; i++) {
        for(int j = 0; j < length; j++) {
          for(int v = -1; v <= 1; v ++) {
            for(int u = -1; u <= 1; u++) {
              if((v + i) >= 0  &&  (v + i ) < width && (u + j) >= 0 && (u + j) < length){
                  buffer[0] = *(source_bitmap + bytes_per_pixel * ((j + u) * width + (i + v)) + 2);
                  buffer[1] = *(source_bitmap + bytes_per_pixel * ((j + u) * width + (i + v)) + 1);
                  buffer[2] = *(source_bitmap + bytes_per_pixel * ((j + u) * width + (i + v)) + 0);
                  buffer[3] = 0;
              }
              else{
                  buffer[0] = 0;
                  buffer[1] = 0;
                  buffer[2] = 0;
                  buffer[3] = 0;
              }
              write_data_to_ACC(FILTER_START_ADDR[hart_id], buffer, 4, hart_id);
              if(hart_id == 0) continue;
              // printf("core%d: send %d %d %d\n", hart_id, *buffer, *(buffer+1), *(buffer+2));
            }
          }
          read_data_from_ACC(FILTER_READ_ADDR[hart_id], buffer, 4, hart_id);
          if(hart_id == 0) continue;
          // printf("core%d: get %d %d %d\n", hart_id, *buffer, *(buffer+1), *(buffer+2));
        }
      }

### Filter code : 進行卷積 (platform/GaussFilter.h)

      void do_filter(){

          wait(CLOCK_PERIOD, SC_NS);

          while (true) {
              int r_val = 0;
              int g_val = 0;
              int b_val = 0;
              for (unsigned int v = 0; v < MASK_Y; ++v) {
                  for (unsigned int u = 0; u < MASK_X; ++u) {
                      unsigned char r = i_r.read();
                      unsigned char g = i_g.read();
                      unsigned char b = i_b.read();
                      wait(CLOCK_PERIOD, SC_NS);
                      r_val += r * mask[u][v];
                      g_val += g * mask[u][v];
                      b_val += b * mask[u][v];
                      wait(CLOCK_PERIOD, SC_NS);
                  }
              }
              o_r.write(r_val / 16);
              o_g.write(g_val / 16);
              o_b.write(b_val / 16);
          }
      }


### 心得

#### 本次作業的 platform/GaussFilter.h(sc_module) 幾乎沒有修改，基本上是和上次作業雷同。src/gauss_multi_core/main.cpp(testbench) 進行圖片的切割的語法也算是簡單，但是在環境的架構上(platform/mc_main.cpp) 還蠻容易出錯。


