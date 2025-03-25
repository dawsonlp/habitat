# habitat
This is a set of collaborating components which together make it easy to manage, develop, use and migrate MCP servers both locally and on the net.

# MCP Habitat: Component Architecture

## Overview

MCP Habitat provides a comprehensive architecture for managing Model Context Protocol (MCP) servers across local and cloud environments. This document outlines the key components and their interactions.

## Core Components

### 1. MCP Registry

The central discovery service implemented as an MCP server itself.

- **Purpose**: Service discovery, authentication, and orchestration
- **Implementation**: MCP server with registry-specific extensions
- **Responsibilities**:
  - Maintain registry of available MCP servers
  - Handle service discovery requests
  - Manage authentication and authorization
  - Route client requests to appropriate MCP servers

### 2. MCP Server Architecture (Three-Tier)

Each MCP server consists of three distinct layers:

#### 2.1 Common MCP Core

- **Purpose**: Handle MCP protocol and conversation management
- **Implementation**: Shared library used by all MCP servers
- **Responsibilities**:
  - Context window management
  - Conversation state tracking
  - Message handling and routing
  - Token counting and limits
  - MCP protocol implementation

#### 2.2 Metadata Service

- **Purpose**: Define server capabilities and integration details
- **Implementation**: Configuration layer for MCP servers
- **Responsibilities**:
  - Define available commands and syntax
  - Provide service schemas
  - Manage versioning information
  - Handle registry registration
  - Document capabilities

#### 2.3 Service Adapter

- **Purpose**: Connect to underlying services (Jira, GitHub, etc.)
- **Implementation**: Independent microservice
- **Responsibilities**:
  - Interface with external APIs
  - Translate between MCP protocol and service-specific APIs
  - Handle service authentication
  - Manage data transformations
  - Provide standalone API access

### 3. Habitat CLI

Command-line interface for managing the entire ecosystem.

- **Purpose**: Unified management interface
- **Implementation**: CLI tool with habitat, registry, and server subcommands
- **Responsibilities**:
  - Manage entire habitat lifecycle
  - Configure and control the registry
  - Add/remove/update individual MCP servers
  - Monitor system health
  - Provision new services

## Interactions

### Registration Flow

1. MCP Server starts up
2. Metadata Service connects to Registry
3. Server authenticates and registers its capabilities
4. Registry adds server to available services
5. Registry performs health checks to ensure availability

### Discovery Flow

1. Client connects to Registry using MCP protocol
2. Client requests available services of a specific type
3. Registry authenticates client and checks permissions
4. Registry returns list of matching services with connection details
5. Client connects directly to relevant MCP Server

### Service Interaction Flow

1. Client establishes conversation with MCP Server
2. Client sends MCP-formatted requests 
3. MCP Core manages conversation context
4. Service Adapter translates requests to service-specific API calls
5. Service Adapter returns data which MCP Core formats as responses

## Deployment Models

### Local Development

- Registry and MCP servers run in local Docker containers
- Services communicate over localhost network
- Configuration stored in local filesystem

### Hybrid Deployment

- Registry runs locally, some services locally, others in cloud
- Local registry maintains connections to remote services
- Consistent discovery mechanism regardless of service location

### Cloud Deployment

- All components deployed to Kubernetes or similar platform
- Registry scales horizontally for high availability
- Service Adapters scale independently based on load
- MCP Servers distributed across regions as needed

## Security Model

### Authentication Layers

1. **Client-to-Registry**: Initial authentication using API keys or OAuth
2. **Registry-to-Server**: Server validation using mutual TLS or API keys
3. **Client-to-Server**: Token-based access provided by Registry
4. **Server-to-Service**: Service-specific authentication handled by Adapter

### Authorization Model

- Registry maintains access control policies
- Permissions managed at service level
- Client capabilities restricted based on identity
- Audit logs capture all authentication and authorization events
