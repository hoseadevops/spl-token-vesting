FROM rust:1.57.0

ARG SOLANA_VERSION=1.9.1
# Latest Release: https://github.com/solana-labs/solana/releases/latest

# Url for Solana Cluster "Network"

# Devnet: https://api.devnet.solana.com
# Testnet: https://api.testnet.solana.com
# Mainnet Beta: https://api.mainnet-beta.solana.com

# Update package manager and install system packages  
RUN apt-get update && \
    apt-get install -y curl

WORKDIR /src/rust/solana

# Install & Configure Solana Core
RUN sh -c "$(curl -sSfL https://release.solana.com/v$SOLANA_VERSION/install)"

CMD [ "/bin/bash" ]