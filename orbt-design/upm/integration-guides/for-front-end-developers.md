# For Front-end Developers

Basic integration:

```javascript
// 1. Import UPM contract
import { UPM_ADDRESS, UPM_ABI } from '@orbt/contracts';

// 2. Encode call data
const callData = orbtUCE.interface.encodeFunctionData('swapExactIn', [
  WBTC_ADDRESS,
  OXBTC_ADDRESS,
  amountIn,
  userAddress,
  0
]);

// 3. Execute via UPM
const tx = await upm.doCall(ORBTUCE_ADDRESS, callData);
await tx.wait();
```

Batch calls:

```javascript
const targets = [ORBTUCE_ADDRESS, S0XBTC_ADDRESS];
const datas = [
  orbtUCE.interface.encodeFunctionData('swapExactIn', [...]),
  sOxBTC.interface.encodeFunctionData('deposit', [...])
];

const tx = await upm.doBatchCalls(targets, datas);
await tx.wait();
```

Error handling:

```javascript
try {
  const tx = await upm.doBatchCalls(targets, datas);
  await tx.wait();
  console.log('Success!');
} catch (error) {
  if (error.message.includes('UPM/len-mismatch')) {
    console.error('Targets and datas length mismatch');
  } else if (error.message.includes('revert')) {
    console.error('One of the calls reverted:', error);
  } else {
    console.error('Unknown error:', error);
  }
}
```
