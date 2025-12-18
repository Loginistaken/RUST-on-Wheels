# RUST-on-Wheels
[package]
name = "rust_on_wheels"
version = "0.1.0"
edition = "2021"

[dependencies]
clap = { version = "4.5.20", features = ["derive"] }
inquire = "0.7.5"
sha2 = "0.10.8"
ed25519-dalek = "2.1.1"
rand = { version = "0.8.5", features = ["getrandom"] }
tokio = { version = "1.41.0", features = ["full"] }
serde = { version = "1.0.210", features = ["derive"] }
serde_json = "1.0.128"
use clap::Parser;
use inquire::{Select, Text};
use rand::rngs::OsRng;
use ed25519_dalek::{Keypair, Signer};
use sha2::{Sha256, Digest};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::io::{self, Write};
use tokio::runtime::Runtime;

// Define modular crypto tasks as enums for recommendations
#[derive(Clone, Copy, Debug, PartialEq)]
enum CryptoTask {
    BlockchainSetup,
    WalletGeneration,
    TransactionSigning,
    ConsensusPoW,
    ConsensusPoS,
}

// Serializable module for LEGO upgrades
#[derive(Serialize, Deserialize, Clone)]
struct CryptoModule {
    code: String,
    explanation: String,
    complexity: u8, // 1-10
    success_rate: u8, // 0-100%
}

// CLI Args
#[derive(Parser)]
#[command(name = "Rust on Wheels")]
#[command(about = "AI-guided Rust crypto coin builder")]
struct Cli {
    #[arg(long)]
    build_coin: bool,
}

fn main() {
    let cli = Cli::parse();

    if cli.build_coin {
        println!("Welcome to Rust on Wheels! Guided crypto dev in Rust.");
        println!("Inspired by Rails, but for Rust crypto: Secure, fast, modular.");
        println!("We'll guide you from basics to advanced, with recs based on real projects like Solana/Polkadot.");

        let rt = Runtime::new().unwrap();
        rt.block_on(build_coin_flow());
    } else {
        println!("Use --build-coin to start.");
    }
}

// Async main flow for potential network sim
async fn build_coin_flow() {
    let mut progress: Vec<CryptoTask> = vec![];
    let mut diagnostics_score = 0;

    loop {
        // Left Panel: Diagnostics
        diagnostics_score = calculate_diagnostics(&progress);
        println!("\n[Diagnostics: {}% Complete] - Add more modules for 100%!", diagnostics_score);

        // Recommendation Pop-ups
        let available_tasks = get_recommended_tasks(&progress);
        if available_tasks.is_empty() {
            println!("Coin build complete! Exporting full code...");
            export_full_code(&progress);
            break;
        }

        // User selects task (simulates clickable rec)
        let task = Select::new("Select next best task (ordered by rec):", available_tasks)
            .with_help_message("Top: Best based on stats/historical success.")
            .prompt()
            .unwrap();

        // Generate and insert code
        let module = generate_module(&task);
        println!("\nGenerated Code Block:\n{}", module.code);
        println!("\nLayman's Explanation: {}", module.explanation);
        println!("Complexity: {}/10 | Success Rate: {}% (from historical crypto projects)", module.complexity, module.success_rate);

        progress.push(task);

        // Bottom Interaction Bar
        interaction_bar().await;
    }
}

// Recommendations with alternatives/stats (AI-simulated)
fn get_recommended_tasks(progress: &Vec<CryptoTask>) -> Vec<String> {
    let mut recs: HashMap<CryptoTask, (u8, String)> = HashMap::new(); // (priority, desc)

    if !progress.contains(&CryptoTask::BlockchainSetup) {
        recs.insert(CryptoTask::BlockchainSetup, (1, "Start with Blockchain Ledger (Success: 95%, Used in Solana for fast ledgers)".to_string()));
    }
    if !progress.contains(&CryptoTask::WalletGeneration) {
        recs.insert(CryptoTask::WalletGeneration, (2, "Wallet Gen (Success: 90%, Like rust-bitcoin HD wallets)".to_string()));
    }
    if !progress.contains(&CryptoTask::TransactionSigning) && progress.contains(&CryptoTask::WalletGeneration) {
        recs.insert(CryptoTask::TransactionSigning, (3, "Transaction Signing (Success: 85%, Ed25519 for secure sigs)".to_string()));
    }
    if !progress.contains(&CryptoTask::ConsensusPoW) && progress.contains(&CryptoTask::BlockchainSetup) {
        recs.insert(CryptoTask::ConsensusPoW, (4, "PoW Consensus (Success: 90%, Bitcoin-style, energy-intensive but proven)".to_string()));
        recs.insert(CryptoTask::ConsensusPoS, (5, "Alt: PoS (Success: 80%, More efficient, like Polkadot Substrate)".to_string())); // Alternative
    }

    // Sort by priority (best at top)
    let mut sorted: Vec<_> = recs.into_iter().collect();
    sorted.sort_by_key(|&(_, (pri, _))| pri);
    sorted.into_iter().map(|(task, (_, desc))| format!("{:?} - {}", task, desc)).collect()
}

// Generate full code paragraph for task
fn generate_module(task: &CryptoTask) -> CryptoModule {
    match task {
        CryptoTask::BlockchainSetup => CryptoModule {
            code: r#"
#[derive(Clone, Debug, Serialize, Deserialize)]
struct Block {
    index: u32,
    timestamp: u64,
    data: String,
    previous_hash: String,
    hash: String,
    nonce: u64,
}

impl Block {
    fn new(index: u32, data: String, previous_hash: String) -> Self {
        let mut block = Block {
            index,
            timestamp: std::time::SystemTime::now().duration_since(std::time::UNIX_EPOCH).unwrap().as_secs(),
            data,
            previous_hash,
            hash: String::new(),
            nonce: 0,
        };
        block.mine(); // For PoW, adjustable difficulty
        block
    }

    fn calculate_hash(&self) -> String {
        let mut hasher = Sha256::new();
        hasher.update(format!("{}{}{}{}{}", self.index, self.timestamp, self.data, self.previous_hash, self.nonce).as_bytes());
        format!("{:x}", hasher.finalize())
    }

    fn mine(&mut self) {
        self.hash = self.calculate_hash();
        while !self.hash.starts_with("0000") { // Simple difficulty
            self.nonce += 1;
            self.hash = self.calculate_hash();
        }
    }
}

#[cfg(feature = "legacy")]
fn legacy_block() { /* Backward compat version */ }
            "#.to_string(),
            explanation: "This sets up a basic blockchain ledger. Rust's strong typing ensures no invalid blocks slip in – safer than C++ in historical crypto bugs. Used in projects like rust-bitcoin for ledgers. Potential: Add ZK-SNARKs for privacy. Add-on friendly: Crate-based, swap hashing algo easily for upgrades.".to_string(),
            complexity: 4,
            success_rate: 95,
        },
        CryptoTask::WalletGeneration => CryptoModule {
            code: r#"
use ed25519_dalek::{Keypair, Signer};
use rand::rngs::OsRng;

fn generate_wallet() -> Keypair {
    let mut csprng = OsRng {};
    Keypair::generate(&mut csprng)
}

let wallet = generate_wallet();
println!("Public Key: {:?}", wallet.public);
            "#.to_string(),
            explanation: "Generates a secure wallet with keys. Rust's crypto crates (like ed25519-dalek) make this memory-safe, preventing leaks that plagued early Bitcoin wallets. Historical: Powers Solana accounts. Future: Quantum-resistant with BLS. Modular: Export to JSON for team collab/hybrids.".to_string(),
            complexity: 3,
            success_rate: 90,
        },
        CryptoTask::TransactionSigning => CryptoModule {
            code: r#"
let wallet = generate_wallet(); // From prior module
let message = b"Transaction: Send 1 coin";
let signature = wallet.sign(message);
println!("Signature: {:?}", signature);
            "#.to_string(),
            explanation: "Signs transactions securely. Rust's borrow checker ensures no double-signing errors. Used in Polkadot for cross-chain tx. Complexity ramps: Next, add multi-sig for advanced. Backward compat: Feature flag for old sig schemes.".to_string(),
            complexity: 5,
            success_rate: 85,
        },
        CryptoTask::ConsensusPoW => CryptoModule {
            code: r#"// Integrate with Block::mine() from blockchain module
// Hybrid upgrade: Swap for PoS by changing impl
#[cfg(feature = "pos_upgrade")]
mod pos { /* PoS logic here */ }
            "#.to_string(),
            explanation: "Proof-of-Work consensus. Proven in Bitcoin (Rust ports exist). Efficient in Rust via concurrency. Alt: PoS for energy savings (80% success in modern chains like Ethereum post-merge). LEGO: Plug into blockchain module for team extensions.".to_string(),
            complexity: 7,
            success_rate: 90,
        },
        CryptoTask::ConsensusPoS => CryptoModule {
            code: r#"// Simplified PoS
fn select_validator(stake: u64) -> bool { stake > 1000 } // Real: Random based on stake
            "#.to_string(),
            explanation: "Proof-of-Stake alt. More scalable than PoW; used in Substrate (Polkadot). Rust's async (Tokio) handles validator networks. Future: AI-optimized staking. Redesign: Manipulate for hybrid (PoW+PoS) compat.".to_string(),
            complexity: 8,
            success_rate: 80,
        },
    }
}

// Diagnostics: Simple % based on completed tasks
fn calculate_diagnostics(progress: &Vec<CryptoTask>) -> u8 {
    let total_tasks = 5; // Expandable
    (progress.len() as u8 * 100 / total_tasks).min(100)
}

// Export full assembled code (for 100%)
fn export_full_code(progress: &Vec<CryptoTask>) {
    let mut full_code = String::new();
    for task in progress {
        let module = generate_module(task);
        full_code.push_str(&module.code);
        full_code.push_str("\n");
    }
    println!("\nFull Assembled Coin Code:\n{}", full_code);
    // Write to file for team collab
    std::fs::write("my_coin.rs", full_code).unwrap();
    println!("Exported to my_coin.rs – Modular, upgradeable!");
}

// Bottom Interaction Bar: Ask questions (restricted to topic)
async fn interaction_bar() {
    loop {
        let query = Text::new("[Interaction Bar] Ask about Rust/crypto coding (or 'exit'):")
            .prompt()
            .unwrap();

        if query.to_lowercase().contains("exit") {
            break;
        }
        if !query.to_lowercase().contains("rust") && !query.to_lowercase().contains("crypto") {
            println!("Stay on topic: Rust/crypto only!");
            continue;
        }

        // Simulated AI response (real: API call)
        println!("Answer: For '{}', here's an example...", query);
        if query.contains("wallet") {
            println!("Best pattern: Use ed25519-dalek. Pros: Secure; Cons: Learning curve. Snippet: [code above]");
            println!("Why: Rust prevents exploits seen in old C wallets. Historical: Solana uses similar for speed.");
        } else if query.contains("consensus") {
            println!("Pros/Cons: PoW secure but energy-hungry; PoS efficient. Rec: PoS for modern coins.");
            println!("Potential: Integrate with Tokio for async validators.");
        } else {
            println!("General: Rust excels in crypto for safety. Ask specifics!");
        }
    }
}
// src/lib.rs
pub mod crypto_utils {
    use sha2::Sha256;
    use sha2::Digest;

    pub fn hash_data(data: &str) -> String {
        let mut hasher = Sha256::new();
        hasher.update(data.as_bytes());
        format!("{:x}", hasher.finalize())
    }
}
