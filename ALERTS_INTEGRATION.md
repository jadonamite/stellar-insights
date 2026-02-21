# Alert System Integration Guide

## Backend Integration

### 1. Add to main.rs or lib.rs:
```rust
mod alerts;
mod alert_handlers;
mod monitor;

use alerts::AlertManager;
use monitor::CorridorMonitor;

// In your main function or app setup:
let (alert_manager, _) = AlertManager::new();
let alert_manager = Arc::new(alert_manager);

// Start monitoring
let monitor = Arc::new(CorridorMonitor::new(
    alert_manager.clone(),
    cache.clone(),
    rpc_client.clone(),
));
tokio::spawn(async move { monitor.start().await });

// Add WebSocket route
.route("/ws/alerts", get(alert_handlers::alert_websocket_handler))
.with_state(alert_manager)
```

### 2. Add dependencies to Cargo.toml:
```toml
tokio = { version = "1", features = ["full"] }
futures = "0.3"
```

## Frontend Integration

### Add to your main layout or App component:
```tsx
import AlertNotifications from '@/components/AlertNotifications';

export default function Layout() {
  return (
    <>
      <AlertNotifications />
      {/* rest of your app */}
    </>
  );
}
```

## Alert Thresholds

- **Success Rate Drop**: Triggers when rate drops >10%
- **Latency Increase**: Triggers when latency increases >50%
- **Liquidity Decrease**: Triggers when liquidity drops >30%

## Monitoring Interval

Corridors are checked every 60 seconds.
