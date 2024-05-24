<template>
  <div>
    <v-card>
      <v-row justify="center">
        <v-col cols="12" sm="10" md="8" lg="6">
          <v-card class="container-card text-center">
            <v-alert dense dark color="green">
              <strong>POC for QR Code Scanner</strong>
            </v-alert>

            <div class="my-3 mx-3">
              <div v-if="showMessage" class="alreadyScanned">
                {{ msg }}
              </div>
            </div>

            <h1>{{ scannedQrCodes }}</h1>
            <h4 class="mt-1">DATE: {{ currentDate }} - TIME: {{ currentTime }}</h4>

            <div class="table-responsive">
              <table class="TableHead">
                <tr>
                  <th>Employee Name</th>
                  <th>Employee ID</th>
                  <th>Case</th>
                  <th>Cost Center</th>
                  <th>DIO</th>
                  <th>Asset Number</th>
                  <th>Item</th>
                  <th>Status</th>
                  <th>Deployed By</th>
                  <th>Claimed</th>
                  <th>Time</th>
                </tr>
                <tr v-for="(mssg, index) in message" :key="index">
                  <td class="mt-1 ml-1">{{ mssg.name }}</td>
                  <td class="mt-1 ml-1">{{ mssg.id }}</td>
                  <td class="mt-1 ml-1">{{ mssg.case }}</td>
                  <td class="mt-1 ml-1">{{ mssg.costCenter }}</td>
                  <td class="mt-1 ml-1">{{ mssg.DIO }}</td>
                  <td class="mt-1 ml-1">{{ mssg.assetNumber }}</td>
                  <td class="mt-1 ml-1">{{ mssg.item }}</td>
                  <td class="mt-1 ml-1">{{ mssg.status }}</td>
                  <td class="mt-1 ml-1">{{ mssg.deployedBy }}</td>
                  <td class="mt-1 ml-1">{{ mssg.claimed }}</td>
                  <td class="mt-1 ml-1">{{ mssg.time }}</td>
                </tr>
              </table>
            </div>

            <div class="d-flex justify-center my-4">
              <v-btn color="primary" @click="showModal = true">Scan QR Code</v-btn>
            </div>

            <v-dialog v-model="showModal" max-width="500px">
              <v-card>
                <v-card-title class="headline">Scan QR Code</v-card-title>
                <v-card-text>
                  <div id="qr-code-full-region"></div>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="primary" text @click="showModal = false">Close</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-card>
        </v-col>
      </v-row>
    </v-card>
  </div>
</template>

<script>
import { Html5Qrcode } from 'html5-qrcode';

export default {
  data() {
    return {
      scannedQrCodes: '',
      currentDate: '',
      currentTime: '',
      message: [],
      showMessage: false,
      msg: false,
      showModal: false,
    };
  },
  methods: {
    createScanQRCodes() {
      const html5QrCodes = new Html5Qrcode("qr-code-full-region");
      const config = { fps: 10, qrbox: { width: 250, height: 250 } };
      html5QrCodes.start({ facingMode: "environment" }, config, this.onScanSuccess);
    },
    createScanQRCodes() {
      this.$nextTick(() => {
        this.html5QrCode = new Html5Qrcode("qr-code-full-region");
        const config = { fps: 10, qrbox: { width: 250, height: 250 } };
        this.html5QrCode.start({ facingMode: "environment" }, config, this.onScanSuccess);
      });
    },
    onScanSuccess(decodeResult) {
      const obj = { decodeResult: decodeResult };
      const existingItem = this.message.find((item) => item.name === this.name(obj.decodeResult));
      if (existingItem) {
        this.showMessage = true;
        this.msg = "Already Scanned";
        setTimeout(() => {
          this.showMessage = false;
        }, 1500);
      } else {
        const currentTime = new Date().toLocaleTimeString([], {
          hour: "2-digit",
          minute: "2-digit",
        });
        const scannedData = this.parseQRCodeData(obj.decodeResult);
        this.message.unshift({
          name: scannedData.employeeName,
          id: scannedData.employeeId,
          case: scannedData.case,
          costCenter: scannedData.costCenter,
          DIO: scannedData.DIO,
          assetNumber: scannedData.assetNumber,
          item: scannedData.item,
          status: scannedData.status,
          deployedBy: scannedData.deployedBy,
          claimed: scannedData.claimed,
          time: currentTime,
        });
        this.showMessage = true;
        this.msg = 'Successfully Scanned';
        setTimeout(() => {
          this.showMessage = false;
        }, 1500);
        this.showModal = false; // Close the modal after successful scan
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
    // name(decodeResult) {
    //   const regex = / - ([^-]+) - /;
    //   const match = decodeResult.match(regex);
    //   return match ? match[1].trim() : "";
    // },
    // id(decodeResult) {
    //   const regex = / - (\d+) /;
    //   const match = decodeResult.match(regex);
    //   return match ? match[1].trim() : "";
    // },
    parseQRCodeData(decodeResult) {
      const parts = decodeResult.split(" - ");
      return {
        employeeName: parts[0],
        employeeId: parts[1],
        case: parts[2],
        costCenter: parts[3],
        DIO: parts[4],
        assetNumber: parts[5],
        item: parts[6],
        status: parts[7],
        deployedBy: parts[8],
        claimed: parts[9],
      };
    },
  },
  watch: {
    showModal(newValue) {
      if (newValue) {
        this.createScanQRCodes();
      } else {
        if (this.html5QrCode) {
          this.html5QrCode.stop();
        }
      }
    },
  },
  mounted() {
    this.updateDate();
    this.updateTime();
  },
};
</script>

<style>
.container-card {
  width: 100%;
  max-width: 800px; /* Adjust the maximum width as needed */
}

.alreadyScanned {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fffffffb;
  color: rgb(11, 231, 11);
  padding: 10px;
  border-radius: 5px;
  z-index: 9999;
}

.table-responsive {
  overflow-x: auto;
}

.TableHead {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1em;
  table-layout: auto;
}

.TableHead th,
.TableHead td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
  word-wrap: break-word;
}

.TableHead th {
  background-color: #f2f2f2;
}

.TableHead tr:nth-child(even) {
  background-color: #f9f9f9;
}

.TableHead tr:hover {
  background-color: #f5f5f5;
}
</style>