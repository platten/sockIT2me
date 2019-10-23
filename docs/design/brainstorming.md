# Initial Brainstorming

## Requirements:
* Portability across *NIX platforms
* ALL traffic encrypted


## Capabilities:
* Distributed service discovery, identity and policy 
* Network connection brokering with the following features:
  * Support for arbitrary UDP and TCP workloads
  * Encryption with mTLS
  * Policy driven AuthZ

## Components:
* Single statically compiled connection broker and control plane daemon
* Client and Server Libraries

## Proposed Address schema:
<scope>.<app group>.<appIdentity>.<resource Group>.<machineName>
**Example:**
`dev.nginxServers.nginx123.default.node1`

## Client:
* Request unix socket based on ID in library
* Bind to socket
* Take input from stdin and send to Open unix pipe
* Receive response
* Log response
* exit

## Server:
* Prepare ID (name of service, host IP)
* Request listening unix socket based on ID in library
* Bind to socket
* Print whatever is received
* Receive message
* message


## Library:
* Request client socket with ID
    * When socket is created send hello
* Request server socket with ID
* Server for health check from server (send I

## Daemon (distributed KV store w/ health check):
* Requirements:
    * Gather configuration
        * Machine identity
        * TTL / keepalive
    * Establish Unix Domain Sockets for local and remote identities
    * local network identity
        * Machine name from parameter/config
        * IP address of machine
        * Key
    * Expose workload network API over local socket connection to:
        * Register workload
            * Purpose: register a workload with the server
            * Input:
                * Identifier
                * Scope
                * From process UID: https://github.com/spiffe/spire/blob/1ec30142daa3a6e9f887925b866e1dade7b6c52e/pkg/common/peertracker/uds_linux.go
            * Output:
                * Return crypto-hash token for identity (local in memory)
                * Return pipe
            * Actions:
                * Create hash of identity
                    * Identity (SPIFFE)
                    * Application name
                    * Scope
                    * PID
                    * User ID
                    * Time registered
                    * Machine Identity
                    * Policy Whitelist
                * Create identity hash token
                * Create pipe to local listening workloads
                    * https://github.com/ohfill/secat/blob/master/secat.go
                * Create identity, endpoint/destination and hash, record in DB with TTL
                * Initiate replication for identity, destination to P2P nodes
                * Get acknowledgement of successful replication to all available nodes

                * Respond
        * Deregister workload
                * Input:
                    * Token
                * Output:
                    * OK
                * Actions:
                    * Delete token from all remote storage endpoints
                    * Delete local identifier
                    * Delete token
                    * Respond
        * Establish connection
            * Input:
                * Session Token
                * Destination:
                    * Application Identity
                    * Machine or runtime group
                        * Multi-path TCP/UDP for runtime groups or client side load balancing
                    * Scope
                    * Protocol (TCP or UDP)
            * Output:
                * Errors:
                    * Timeout
                        * Why: Source Workload Session expired / token cleared
                    * Invalid Source Workload:
                        * Why: Source Workload not registered
                    * Destination not found
                        * Why: Destination not found in data store
                    * Destination unreachable
                        * Why: Cannot connect to destination host? Is the host up?
                        * Side effect: trigger check and potential cleanup of destination host from workload db 
                    * Cannot connect
                        * Why: cannot set up Unix Domain Socket
                    * Unauthorized:
                        * Why: connectivity is not authorized due to differing scope or lack of whitelist in policy
                * Success:
                    * Path to Unix socket with UID same as Source
            * Actions:
                * Check if session token is valid
                    * Return error if not, workload will have to re-auth
                * Check if source workload is valid
                    * Check if source workload exists
                        * If not in store, then invalid source workload (need to register)
                    * Check if source workload is running
                        * Delete source sockets
                        * Remove source socket references from store
                        * Delete source workload identity
                        * Propagate deletion
                        * Remove token
                        * Return error (need to re-register)
                    * Check if destination exists:
                        * Check if destination IP and destination endpoint is in store, if not then destination not found
                    * Check if can ping destination IP
                        * Check if can ping daemon on destination, if not then Destination Unreachable
                    * Attempt to create socket with source UID to destination socket proxied over TCP/UDP using netcat like pipe
                        * If not running clear out source identity
        * Set point to point network socket connection
    * Espose 
    * Ability to bootstrap and synchronize data
    * Ability to create local and remote sockets
        * Potential support for multipath UDP and TCP
        * Should implement with zeromq
    * 
    * Receive message for creating local socket
* Read:
    * Read socket directory target
    * Read machine identity
    * Socket to bind to
    * Address of other servers
* Store machine identity
* Listen for request
    * Get request with payload IDentity, IP (default local IP), protocol (TCP/UDP) should be received
        * Figure out if socket is for local 
        * Generate random unused socket
    * Should respond with random socket address path
    * Wait for hello packet from client
