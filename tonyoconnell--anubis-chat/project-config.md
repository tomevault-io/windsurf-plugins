---
trigger: always_on
description: ComputeBudgetProgram,
---


# Transaction Handling & Error Recovery

## Transaction Lifecycle Management

### 1. Transaction Builder with Optimization
```typescript
import { 
  Transaction, 
  SystemProgram, 
  ComputeBudgetProgram,
  PublicKey,
  TransactionInstruction,
  Connection,
  SendOptions
} from '@solana/web3.js';
import { AnchorWallet } from '@solana/wallet-adapter-react';

export class OptimizedTransactionBuilder {
  private instructions: TransactionInstruction[] = [];
  private connection: Connection;
  private wallet: AnchorWallet;
  private priorityFee: number = 0;
  private computeUnitLimit: number = 200000;

  constructor(connection: Connection, wallet: AnchorWallet) {
    this.connection = connection;
    this.wallet = wallet;
  }

  public addInstruction(instruction: TransactionInstruction): this {
    this.instructions.push(instruction);
    return this;
  }

  public setPriorityFee(microLamports: number): this {
    this.priorityFee = microLamports;
    return this;
  }

  public setComputeUnitLimit(units: number): this {
    this.computeUnitLimit = units;
    return this;
  }

  public async build(): Promise<Transaction> {
    const transaction = new Transaction();

    // Add compute budget instructions if specified
    if (this.priorityFee > 0) {
      transaction.add(
        ComputeBudgetProgram.setComputeUnitPrice({
          microLamports: this.priorityFee
        })
      );
    }

    if (this.computeUnitLimit !== 200000) {
      transaction.add(
        ComputeBudgetProgram.setComputeUnitLimit({
          units: this.computeUnitLimit
        })
      );
    }

    // Add all instructions
    transaction.add(...this.instructions);

    // Set recent blockhash and fee payer
    const { blockhash, lastValidBlockHeight } = await this.connection.getLatestBlockhash();
    transaction.recentBlockhash = blockhash;
    transaction.feePayer = this.wallet.publicKey;

    return transaction;
  }

  public async estimateComputeUnits(): Promise<number> {
    try {
      const transaction = await this.build();
      
      // Simulate transaction to get compute unit usage
      const simulationResult = await this.connection.simulateTransaction(transaction);
      
      if (simulationResult.value.err) {
        console.warn('Transaction simulation failed:', simulationResult.value.err);
        return this.computeUnitLimit;
      }

      const computeUnitsUsed = simulationResult.value.unitsConsumed || 200000;
      
      // Add 20% buffer for safety
      return Math.ceil(computeUnitsUsed * 1.2);
    } catch (error) {
      console.error('Failed to estimate compute units:', error);
      return this.computeUnitLimit;
    }
  }

  public async estimateOptimalPriorityFee(): Promise<number> {
    try {
      // Get recent priority fees from successful transactions
      const recentPriorityFees = await this.connection.getRecentPrioritizationFees();
      
      if (recentPriorityFees.length === 0) {
        return 0;
      }

      // Calculate 75th percentile for competitive fees
      const sortedFees = recentPriorityFees
        .map(fee => fee.prioritizationFee)
        .sort((a, b) => a - b);
      
      const percentile75Index = Math.floor(sortedFees.length * 0.75);
      return sortedFees[percentile75Index];
      
    } catch (error) {
      console.error('Failed to estimate priority fee:', error);
      return 0;
    }
  }
}
```

### 2. Advanced Transaction Sender
```typescript
export interface TransactionOptions {
  maxRetries?: number;
  retryDelay?: number;
  skipPreflight?: boolean;
  preflightCommitment?: Commitment;
  maxSupportedTransactionVersion?: number;
  minContextSlot?: number;
}

export interface TransactionResult {
  signature: string;
  blockhash: string;
  lastValidBlockHeight: number;
  confirmationStatus: TransactionConfirmationStatus;
  slot?: number;
  err?: TransactionError | null;
}

export class AdvancedTransactionSender {
  private connection: Connection;
  private wallet: AnchorWallet;
  private metricsCollector?: RPCMetricsCollector;

  constructor(
    connection: Connection, 
    wallet: AnchorWallet,
    metricsCollector?: RPCMetricsCollector
  ) {
    this.connection = connection;
    this.wallet = wallet;
    this.metricsCollector = metricsCollector;
  }

  public async sendAndConfirmTransaction(
    transaction: Transaction,
    options: TransactionOptions = {}
  ): Promise<TransactionResult> {
    const {
      maxRetries = 3,
      retryDelay = 1000,
      skipPreflight = false,
      preflightCommitment = 'confirmed'
    } = options;

    const sendOptions: SendOptions = {
      skipPreflight,
      preflightCommitment,
      maxRetries: 0 // We handle retries ourselves
    };

    let lastError: Error | null = null;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      const startTime = Date.now();
      
      try {
        // Sign transaction
        const signedTransaction = await this.wallet.signTransaction(transaction);
        
        // Send transaction
        const signature = await this.connection.sendRawTransaction(
          signedTransaction.serialize(),
          sendOptions
        );

        // Confirm transaction
        const confirmationResult = await this.confirmTransactionWithTimeout(
          signature,
          transaction.recentBlockhash!,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tonyoconnell) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
