rust_on_wheels.rs   ✅ FINAL / AUTHORITATIVE
 use clap::Parser;
use inquire::{Select, Text};
use rand::rngs::OsRng;
use ed25519_dalek::{Keypair, Signer};
use sha2::{Sha256, Digest};
use serde::{Deserialize, Serialize};
use std::collections::{HashMap, HashSet};
use std::fs::File;
use std::io::{self, Read, Write};
use std::net::TcpStream;
use std::process::Command;
use tokio::runtime::Runtime;

// Help template
const HELP_TEXT: &str = r#"
=================================================================
                  RUST ON WHEELS – INTERACTION HELP
=================================================================
Welcome to Rust on Wheels – your AI-guided crypto builder in Rust!
...
Type "help" for this again. "exit" to proceed.
Enjoy – Rust makes crypto safe!
=================================================================
"#;

// Define tasks
#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
enum CryptoTask {
    BlockchainSetup,
    WalletGeneration,
    TransactionSigning,
    ConsensusPoW,
    ConsensusPoS,
    ZkSnarksPrivacy, 
    QuantumResistant, 
}

// Serializable module
#[derive(Serialize, Deserialize, Clone)]
struct CryptoModule {
    code: String,
    explanation: String,
    complexity: u8, 
    success_rate: u8,
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
        println!("Welcome to Rust on Wheels! AI-guided crypto dev in Rust.");
        println!("{}", HELP_TEXT); 

        let rt = Runtime::new().unwrap();
        rt.block_on(build_coin_flow());

        // Auto-send after build
        auto_send_source_16bit("rust_on_wheels.rs", "192.168.1.100", 4000);
    } else {
        println!("Use --build-coin to start.");
    }
}

// Main async flow
async fn build_coin_flow() {
    let mut progress: HashSet<CryptoTask> = HashSet::new();
    let mut diagnostics_score = 0;

    loop {
        diagnostics_score = calculate_diagnostics(&progress);
        println!("\n[Diagnostics: {}% Complete]", diagnostics_score);
        simulated_popup("Complexity Review", "Current avg complexity: 5/10.");

        let available_tasks = get_weighted_recommendations(&progress);
        if available_tasks.is_empty() {
            println!("Coin build complete! Exporting...");
            export_full_code(&progress);
            break;
        }

        let task_str = Select::new("Select next task:", available_tasks.clone())
            .with_help_message("Better choice? We suggest hybrids for max complexity.")
            .prompt()
            .unwrap();
        let task = parse_task(&task_str);

        if progress.contains(&task) {
            println!("Task already done – no repeats!");
            continue;
        }

        if matches!(task, CryptoTask::ConsensusPoW) && progress.contains(&CryptoTask::ConsensusPoS) {
            println!("PoS already chosen – upgrade to hybrid instead?");
            if Text::new("Upgrade to hybrid? (y/n)").prompt().unwrap() == "y" {
                println!("Upgraded to hybrid consensus – max complexity!");
            }
            continue;
        }

        let module = generate_module(&task);
        println!("\nGenerated Code:\n{}", module.code);
        println!("\nExplanation: {}", module.explanation);

        progress.insert(task);

        interaction_bar().await;
    }
}

