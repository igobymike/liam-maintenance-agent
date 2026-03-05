# LIAM — LaPorte Intelligent Asset Management

AI-powered daily maintenance operations agent for industrial plants.

## What it does

- Pulls daily SAP maintenance data automatically
- Filters work by crew: VN-PM (Mechanical) and VN-EC (Electrical/Instrumentation)
- Generates plain-English daily summaries for leadership via email
- Copilot Studio agent for on-demand Q&A about the maintenance schedule

## Architecture

SAP batch export → SharePoint → Power Automate → AI Builder → Email + Copilot Studio agent

## Stack

- SAP (IW47, IP19, Prometheus)
- SharePoint
- Power Automate
- AI Builder (GPT summarization)
- Microsoft Copilot Studio
- Power BI

## Status

In development — targeting production at Kuraray LaPorte plant.

## Author

Mike Birklett — https://baiteks.com
