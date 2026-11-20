# Lab 3: Component Modelling & Architectural Pattern Selection

## Architectural Justification – Self-Service Coffee Kiosk System

**Name:** Shonit Balaguragi  
**Roll No:** PES2UG24AM153

### Architecture Selection

We chose Layered Architecture for the Self-Service Coffee Kiosk System.

### 1. Architectural Choice

A three-layer style: Presentation (User Interface Component), Business (Order Manager, Payment Service and Receipt Printer Components) and Data (Menu Database Component). Layers communicate only through the provided/required interfaces IOrder, IPayment, IReceipt and IMenuData shown in the component diagram.

Microservices was rejected because a single kiosk with three drinks and two sizes does not justify network hops and operational overhead; Client-Server was rejected because the kiosk must keep working at the counter without depending on a central server that would be a single point of failure.

### 2. Two Reasons for the Choice

**Reason 1 – The workflow is small, fixed and sequential.**

Selecting a coffee, choosing a size, paying by card and printing a receipt is a linear flow that maps directly onto a top-down layered design, with the Order Manager coordinating each step. It is simple to build, test and explain.

**Reason 2 – Layers map to the technical constraints.**

The touch screen lives in the Presentation layer, card payment and the receipt-printer hardware are isolated behind Business-layer components, and menu and pricing data are stored in the Data layer. Any one part (a new printer driver, a different payment provider, a new drink added to the menu) can be replaced without changing the others because the interfaces stay the same.

### 3. Security Advantage

The UI never talks to the Payment Service or the database directly; every request passes through the single, controlled IOrder interface of the Order Manager. Card data is handled only inside the Payment Service (sent straight to the payment gateway over TLS, never stored in the Menu Database or shown by the UI), the receipt prints only a masked card number, and the Data layer is reached only through IMenuData using parameterised queries, which limits injection and data exposure.

### 4. Performance Benefit

All components run in-process on the kiosk, so calls between layers are direct method calls with no network latency (unlike microservices), keeping touch-screen response immediate. The small menu and price list can be loaded and cached by the Order Manager at start-up, so each tap does not hit the database; only card authorisation needs the network.

### Interface Summary

| Interface | Provided by | Required by | Technology |
|---|---|---|---|
| IOrder | Order Manager | User Interface | API call |
| IPayment | Payment Service | Order Manager | API call |
| IReceipt | Receipt Printer | Order Manager | Hardware interface |
| IMenuData | Menu Database | Order Manager | SQL query |
