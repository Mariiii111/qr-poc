<template>
  <div>
    <v-card>
      <v-row>
        <v-col cols="12" class="d-flex justify-center">
          <v-card width="500px">
            <v-alert dense dark color="green">
              <strong>POC for QR Code Scanner</strong>
            </v-alert>
            
            <div id="qr-code-full-region" class="my-3 mx-3">
              <div v-if="showMessage" class="alreadyScanned">
                {{ msg }}
              </div>
            </div>

            <h1>{{ scannedQrCodes }}</h1>
            <h4 class="mt-1">DATE: {{ currentDate }} - TIME: {{ currentTime }}</h4>

            <div>
              <table class="TableHead">
                <tr>
                  <th>Employee ID</th>
                  <th>Employee Name</th>
                  <th>Time</th>
                  <!-- <th>Cost Center</th>
                  <th>DIO</th>
                  <th>Asset Number</th>
                  <th>Item</th>
                  <th>Status</th>
                  <th>Deployed By</th>
                  <th>Date Time Deployed</th>
                  <th>Claimed</th> -->
                </tr>
                <tr v-for="(mssg, index) in message" :key="index">
                  <td class="mt-1 ml-1">{{ mssg.id }}</td>
                  <td class="mt-1 ml-1">{{ mssg.name }}</td>
                  <td class="mt-1 ml-1">{{ mssg.time }}</td>
                  <!-- <td style="font-size: 12px;" class="mt-1 ml-1"></td>
                  <td style="font-size: 12px;" class="mt-1 ml-1"></td>
                  <td style="font-size: 12px;" class="mt-1 ml-1"></td>
                  <td style="font-size: 12px;" class="mt-1 ml-1"></td>
                  <td style="font-size: 12px;" class="mt-1 ml-1"></td>
                  <td style="font-size: 12px;" class="mt-1 ml-1"></td>
                  <td style="font-size: 12px;" class="mt-1 ml-1"></td>
                  <td style="font-size: 12px;" class="mt-1 ml-1"></td> -->
                </tr>
              </table>
            </div>
          </v-card>
        </v-col>
      </v-row>
    </v-card>
  </div>
</template>

<script>
  import { Html5Qrcode } from 'html5-qrcode';
  export default {
    data(){
      return{
        scannedQrCodes: '',
        currentDate: '',
        currentTime: '',
        message: [],
        showMessage: false,
        msg: false,
      }
    },
    methods: {
      createScanQRCodes() {
        const html5QrCodes = new Html5Qrcode("qr-code-full-region");
        const config = { fps: 10, qrbox: { width: 250, height: 250 } }
        html5QrCodes.start({ facingMode: "environment" }, config, this.onScanSuccess);
      },
      onScanSuccess(decodeResult) {
        // this.scannedQrCodes = decodeResult;
        const obj = { decodeResult: decodeResult };
        if (this.message.find((item) => item.name == this.name(obj.decodeResult))) {
          this.showMessage = true;
          this.msg = "Already Scanned";
          setTimeout(() => {
            this.showMessage = false;
          }, 1500);
        } else {
          const currentTime = new Date().toLocaleTimeString([], {
            hour: "2-digit",
            minute: "2-digit",
          })
          this.message.unshift({
            name: this.name(obj.decodeResult),
            id: this.id(obj.decodeResult),
            time: currentTime,
          });
          this.showMessage = true;
          this.msg = 'Successfully Scanned';
          setTimeout(() => {
            this.showMessage = false;
          }, 1500);
        }
      },
      updateDate() {
        const now = new Date();
        const options = { year: 'numeric', month: 'long', day: 'numeric' };
        this.currentDate = now.toLocaleDateString(undefined, options);
      },
      updateTime() {
        const now = new Date();
        const hours = now.getHours();
        const minutes = now.getMinutes();
        const ampm = hours >= 12 ? 'PM' : 'AM';
        const formattedHours = hours % 12 || 12;
        this.currentTime = `${formattedHours}:${this.padZero(minutes)} ${ampm}`;
      },
      padZero(num) {
        return num.toString().padStart(2, '0');
      },
      name(decodeResult) {
        const regex = / - ([^-]+) - /;
        const match = decodeResult.match(regex);
        return match ? match[1].trim() : "";
      },
      id(decodeResult) {
        const regex = / - (\d+) /;
        const match = decodeResult.match(regex);
        return match ? match[1].trim() : "";
      },
    },
    mounted() {
      this.createScanQRCodes();
      this.updateDate();
      this.updateTime();
    },
  }
</script>
<style>
  .alreadyScanned {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fffffffb;
    color: rgb(11, 231, 11);
    padding: 10px;
    border-radius: 5px;
    z-index: 9999
  }

  .TableHead {
    width: 100%;
    border-collapse: collapse;
    margin-top: 1em;
  }

  .TableHead th, .TableHead td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }

  .TableHead th {
    background-color: #f2f2f2
  }

  .TableHead tr:nth-child(even) {
    background-color: #f9f9f9;
  }

  .TableHead tr:hover {
    background-color: #f5f5f5;
  }
</style>