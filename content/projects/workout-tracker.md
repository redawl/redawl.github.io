+++
title = 'Workout Tracker'
date = 2024-07-14T11:36:29-07:00
draft = false
+++

## Workout Tracker 
[Source](https://github.com/redawl/Workout-Tracker)

![icon](https://raw.githubusercontent.com/redawl/Workout-Tracker/main/frontend/public/icon.png)
### A simple tracker of workouts!

### [Official Site](https://workout.burchbytes.com)

### Getting Started
Clone the repo:
```bash
$ git clone https://github.com/redawl/Workout-Tracker.git
```

Create Env File:
```bash
$ vim .env
```
```properties
### Postgres ###
POSTGRES_USER={DB_USER}
POSTGRES_PASS={DB_PASS}
POSTGRES_DB={DB}

### Auth Server ###
API_DOMAIN={AUTH_SERVER_IP}
```

Build and start the application:
```bash
$ ./scripts/run-dev.sh
```

That's it!
