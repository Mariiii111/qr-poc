<script>
import { Html5Qrcode } from 'html5-qrcode';

export default {
  data() {
    return {
      modalVisible: false,
      html5QrCodes: null,
    };
  },
  methods: {
    openModal() {
      this.modalVisible = true;
    },
    closeModal() {
      this.modalVisible = false;
      if (this.html5QrCodes) {
        this.html5QrCodes.stop().then(ignore => {
          // QR code scanning stopped
        }).catch(err => {
          console.error(err);
        });
      }
    },
    initializeQrCodeScanner() {
      try {
        const qrCodeFullRegion = document.querySelector('.qr-scanner');
        if (!qrCodeFullRegion) {
          throw new Error('QR code region element not found');
        }
        this.html5QrCodes = new Html5Qrcode(qrCodeFullRegion);
        const config = { fps: 10, qrbox: { width: 250, height: 250 } };
        this.html5QrCodes.start({ facingMode: "environment" }, config, this.onScanSuccess);
      } catch (error) {
        console.error('Failed to initialize QR code scanner:', error);
        // You can show an error message to the user here
      }
    },
    handleModalUpdate(newValue) {
      if (newValue) {
        this.openModal();
        this.$nextTick(() => {
          this.initializeQrCodeScanner();
        });
      } else {
        this.closeModal();
      }
    },
    onScanSuccess(decodeResult) {
      this.$emit('scan-success', decodeResult);
      this.closeModal();
    },
  },
};
</script>
