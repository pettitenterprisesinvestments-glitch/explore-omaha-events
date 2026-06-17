# 🌽 Explore Omaha - Daily Family Events Archive

A daily time machine of family-friendly events in Omaha, Nebraska.

## About
This repository is automatically updated every morning at 7am CT by Superagent.
It tracks family-friendly, faith-based, and conservative-appropriate events for Omaha families.

## Content Standards
All events are filtered to meet the following criteria:
- ✅ Family-friendly (suitable for kids ages 0-12)
- ✅ Faith-based, Christian, or neutral/conservative content
- ✅ Festivals, concerts, restaurants, outdoor activities, community events
- ❌ No LGBTQ content
- ❌ No adult content or profanity
- ❌ No content unsuitable for young children

## Structure
```
/daily/
  YYYY-MM-DD.json   ← One file per day with all events pulled that day
/archive/
  YYYY-MM.csv       ← Monthly CSV archives
README.md
```

## Data Fields
Each event record includes:
- `event_name` - Name of the event
- `event_date` - Date and time of the event
- `location` - Venue and address
- `category` - Festival, Concert, Church/Faith, Outdoor Activity, etc.
- `age_range` - Recommended ages
- `cost` - Free or price range
- `description` - Event details
- `source_url` - Where we found it
- `family_rating` - ⭐⭐⭐ to ⭐⭐⭐⭐⭐
- `is_faith_based` - true/false
- `is_free` - true/false
- `date_pulled` - When this was added

## Connected To
- 📊 [Google Sheet](https://docs.google.com/spreadsheets/d/1YVZwU6PIYq1qGtnMlaiKrUESYjPXzS6tJeJJESTGyOE/edit) — live view of current events
- 🌐 [Explore Omaha Online](https://exploreomaha.online) — the website this feeds

## Powered By
Superagent on Base44 — automated daily at 7:00 AM CT
