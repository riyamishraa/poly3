# zardkat 🐱

A [hardhat-circom](https://github.com/projectsophon/hardhat-circom) template to generate zero-knowledge circuits, proofs, and solidity verifiers

## Quick Start
Compile the riyaCustomCircuit() circuit and verify it against a smart contract verifier

```
pragma circom 2.0.0;

/*This circuit template checks that c is the multiplication of a and b.*/  

template riyaCustomCircuit() {  
  
  //signal inputs

  signal input A;
  signal input B;

  //signals from gates

  signal X;
  signal Y;

  //final signal output

  signal output Q;

  //component gates used to create custom circuit

  component andGate = AND();
  component notGate = NOT();
  component orGate = OR();

  //circuit logic 

  andGate.a <== A;
  andGate.b <== B;
  X <== andGate.out;

  notGate.in <== B;
  Y <== notGate.out;

  orGate.a <== X;
  orGate.b <== Y;
  Q <== orGate.out;
}

template AND() {
    signal input a;
    signal input b;
    signal output out;

    out <== a * b;
}

template NOT() {
    signal input in;
    signal output out;

    out <== 1 + in - 2 * in;
}

template OR() {
    signal input a;
    signal input b;
    signal output out;

    out <== a + b - a * b;
}

component main = riyaCustomCircuit();

```
### Install
`npm i`

### Compile
`npx hardhat circom` 
This will generate the **out** file with circuit intermediaries and geneate the **riyaCustomCircuitVerifier.sol** contract

### Prove and Deploy
`npx hardhat run scripts/deploy.ts`
This script does 4 things  
1. Deploys the riyaCustomCircuit()Verifier.sol contract
2. Generates a proof from circuit intermediaries with `generateProof()`
3. Generates calldata with `generateCallData()`
4. Calls `verifyProof()` on the verifier contract with calldata

With two commands you can compile a ZKP, generate a proof, deploy a verifier, and verify the proof 🎉

## Configuration
### Directory Structure
**circuits**
```
├── riyaCustomCircuit()
│   ├── circuit.circom
│   ├── input.json
│   └── out
│       ├── circuit.wasm
│       ├── riyaCustomCircuit().r1cs
│       ├── riyaCustomCircuit().vkey
│       └── riyaCustomCircuit().zkey
├── new-circuit
└── powersOfTau28_hez_final_12.ptau
```
Each new circuit lives in it's own directory. At the top level of each circuit directory lives the circom circuit and input to the circuit.
The **out** directory will be autogenerated and store the compiled outputs, keys, and proofs. The Powers of Tau file comes from the Polygon Hermez ceremony, which saves time by not needing a new ceremony. 


**contracts**
```
contracts
└── riyaCustomCircuitVerifier.sol
```
Verifier contracts are autogenerated and prefixed by the circuit name, in this example **riyaCustomCircuit**

## hardhat.config.ts
```
  circom: {
    // (optional) Base path for input files, defaults to `./circuits/`
    inputBasePath: "./circuits",
    // (required) The final ptau file, relative to inputBasePath, from a Phase 1 ceremony
    ptau: "powersOfTau28_hez_final_12.ptau",
    // (required) Each object in this array refers to a separate circuit
    circuits: JSON.parse(JSON.stringify(circuits))
  },
```
### circuits.config.json
circuits configuation is separated from hardhat.config.ts for **autogenerated** purposes (see next section)
```
[
  {
    "name": "riyaCustomCircuit",
    "protocol": "groth16",
    "circuit": "riyaCustomCircuit/circuit.circom",
    "input": "riyaCustomCircuit/input.json",
    "wasm": "riyaCustomCircuit/out/circuit.wasm",
    "zkey": "riyaCustomCircuit/out/riyaCustomCircuit.zkey",
    "vkey": "riyaCustomCircuit/out/riyaCustomCircuit.vkey",
    "r1cs": "riyaCustomCircuit/out/riyaCustomCircuit.r1cs",
    "beacon": "0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f"
  }
]
```

**adding circuits**   
To add a new circuit, you can run the `newcircuit` hardhat task to autogenerate configuration and directories i.e  
```
npx hardhat newcircuit --name newcircuit
```

**determinism**
> When you recompile the same circuit using the groth16 protocol, even with no changes, this plugin will apply a new final beacon, changing all the zkey output files. This also causes your Verifier contracts to be updated.
> For development builds of groth16 circuits, we provide the --deterministic flag in order to use a NON-RANDOM and UNSECURE hardcoded entropy (0x000000 by default) which will allow you to more easily inspect and catch changes in your circuits. You can adjust this default beacon by setting the beacon property on a circuit's config in your hardhat.config.js file.

**author**
(Riya Mishra)[https://www.linkedin.com/in/riya-mishra-a2b6ab213/]

