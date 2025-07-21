# Connect via SSH

Learn how to establish a secure connection to RCC systems using SSH with Duo Push authentication.

## Basic connection

Connect to RCC using your university credentials:

```bash
ssh username@midway3.rcc.uchicago.edu
```

Replace `username` with your RCC username. You'll be prompted to:
1. Enter your password
2. Complete Duo Push authentication on your registered device

!!! warning "Two-factor authentication required"
    All RCC connections require Duo Push authentication for security.

## Why midway3?

While you can connect to both `midway2` and `midway3`, **use `midway3` for all new work**:

- More up-to-date system with latest software
- Actively maintained and supported
- Default system for new users
- Better performance and reliability

## Connection workflow

Every RCC login follows this pattern:

1. **Initiate SSH connection** and enter your password
2. **Duo Push notification** appears on your registered device
3. **Approve the request** on your phone/device
4. **Connection established** to midway3

## Next steps

Once connected to midway3, learn how to start [interactive sessions](interactive-sessions.md) for your research work.