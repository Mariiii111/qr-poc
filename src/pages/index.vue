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

            <div v-if="scannedData" class="scanned-data mx-7">
              <h4><strong>{{ scannedData.employeeName }}</strong></h4>
              <p><strong>Employee ID:</strong> {{ scannedData.employeeId }}</p>
              <p><strong>Cost Center:</strong> {{ scannedData.costCenter }}</p>
              <p><strong>Department Inventory Officer:</strong> {{ scannedData.DIO }}</p>
              <p><strong>Asset No. {{ scannedData.assetNumber }}</strong></p>
              <p><strong>Case:</strong> {{ scannedData.case }}</p>
              <p><strong>Item:</strong> {{ scannedData.item }}</p>
              <p><strong>Status:</strong> {{ scannedData.status }}</p>
              <p><strong>Deployed By:</strong> {{ scannedData.deployedBy }} - {{ scannedData.dateTimeDeployed }}</p>
              <p><strong>Claimed:</strong> {{ scannedData.claimed }}</p>
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
      currentDate: '',
      currentTime: '',
      scannedData: null,
      showMessage: false,
      msg: false,
      showModal: false,
    };
  },
  methods: {
    createScanQRCodes() {
      this.$nextTick(() => {
        this.html5QrCode = new Html5Qrcode("qr-code-full-region");
        const config = { fps: 10, qrbox: { width: 250, height: 250 } };
        this.html5QrCode.start({ facingMode: "environment" }, config, this.onScanSuccess);
      });
    },
    onScanSuccess(decodeResult) {
      const obj = { decodeResult: decodeResult };
      const scannedData = this.parseQRCodeData(obj.decodeResult);
      this.scannedData = scannedData;
      this.showMessage = true;
      this.msg = 'Successfully Scanned';
      setTimeout(() => {
        this.showMessage = false;
      }, 1500);
      this.showModal = false;
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
    parseQRCodeData(decodeResult) {
      try {
        const jsonContent = decodeResult.replace(/^export default /, '');
        const data = JSON.parse(jsonContent);
        if (Array.isArray(data) && data.length > 0) {
          return data[0]; // Assuming the QR code contains an array with a single object
        } else {
          return data;
        }
      } catch (error) {
        console.error('Error parsing QR code data:', error);
        return null;
      }
    }
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

.scanned-data {
  text-align: left;
  margin-top: 1rem;
}

.scanned-data p {
  margin-bottom: 0.5rem;
}
</style>