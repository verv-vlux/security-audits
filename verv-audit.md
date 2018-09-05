![frontcover](https://user-images.githubusercontent.com/41786403/45082081-79637180-b0f0-11e8-8e97-6baaa638d40a.png)

## Section 1 - Table of Contents
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

  - [Section 2 - Introduction](#section-2---introduction)
    - [2.1 Authenticity](#21-authenticity)
    - [2.2 Audit Goals and Focus](#22-audit-goals-and-focus)
      - [2.2.1 Sound Architecture](#221-sound-architecture)
      - [2.2.2 Smart Contract Best Practices](#222-smart-contract-best-practices)
      - [2.2.3 Code Correctness](#223-code-correctness)
      - [2.2.4 Code Quality](#224-code-quality)
      - [2.2.5 Security](#225-security)
      - [2.2.6 Testing and testability](#226-testing-and-testability)
    - [2.3 About the Verv Flux ICO](#23-about-the-verv-flux-ico)
    - [2.4 Terminology](#24-terminology)
      - [2.4.1 Likelihood](#241-likelihood)
      - [2.4.2 Impact](#242-impact)
      - [2.4.3 Severity](#243-severity)
  - [Section 3 - Overview](#section-3---overview)
    - [3.1 Source Code](#31-source-code)
    - [3.2 General Notes](#32-general-notes)
    - [3.3 Contracts](#33-contracts)
  - [Section 4 - Testing](#section-4---testing)
  - [Section 5 - Audit findings](#section-5---audit-findings)
    - [5.1 Note Issues](#51-note-issues)
      - [5.1.1 finalizationRetainStrategy[companyWallet] does not need to be a map](#511-finalizationretainstrategycompanywallet-does-not-need-to-be-a-map)
      - [5.1.2 investorsMarge should not need to be calculated at finalize](#512-investorsmarge-should-not-need-to-be-calculated-at-finalize)
      - [5.1.3 updateCap has no check for 20 mil hardcap which is mentioned at comment.](#513-updatecap-has-no-check-for-20-mil-hardcap-which-is-mentioned-at-comment)
      - [5.1.4 wither cap is a typo](#514-wither-cap-is-a-typo)
      - [5.1.5 bonuses variable defined but never used](#515-bonuses-variable-defined-but-never-used)
      - [5.1.6 whenNotPaused modifier is missing on disbursePreBuyersLkdContributions function](#516-whennotpaused-modifier-is-missing-on-disburseprebuyerslkdcontributions-function)
    - [5.2 Low Issues](#52-low-issues)
      - [5.2.1 Vesting period can be manipulated by changing endTime](#521-vesting-period-can-be-manipulated-by-changing-endtime)
      - [5.2.2 updateCap does not check if new cap is more than weiRaised](#522-updatecap-does-not-check-if-new-cap-is-more-than-weiraised)
      - [5.2.3 start and end time are hardcoded on constructor](#523-start-and-end-time-are-hardcoded-on-constructor)
      - [5.2.4 hasEnded defined in super class and sub class which offer different functionality](#524-hasended-defined-in-super-class-and-sub-class-which-offer-different-functionality)
      - [5.2.5 uint for representing stage should be uint8](#525-uint-for-representing-stage-should-be-uint8)
      - [5.2.6 Whitelisting may take too long](#526-whitelisting-may-take-too-long)
    - [5.3 Medium Issues](#53-medium-issues)
    - [5.4 High Issues](#54-high-issues)
      - [5.4.1 Whitelisted users can contribute more than 10 ETH in day 1](#541-whitelisted-users-can-contribute-more-than-10-eth-in-day-1)
    - [5.4.2 Users can contribute more than cap](#542-users-can-contribute-more-than-cap)
    - [5.5 Critical Issues](#55-critical-issues)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## Section 2 - Introduction

Makoto Inoue and Laurence Kirk performed an audit of the Verv Flux Crowdsale smart contracts under
the supervision of Matthew Di Ferrante.

Makoto Inoue and Laurence Kirk examined the source code independently at the beginning, then
discuss the finding to create the joint report. Matthew Di Ferrante reviewed the final report and signed
off.

I, Makoto Inoue have no stake or vested interest in Verv Energy.
I, Laurence Kirk have no stake or vested interest in Verv Energy.
I, Matthew Di Ferrante have no stake or vested interest in Verv Energy.

### 2.1 Authenticity

This document should have an attached cryptographic signature to ensure it has not been tampered
with. The signature can be verified using the public key from [TO BE ADDED]

### 2.2 Audit Goals and Focus

#### 2.2.1 Sound Architecture

This audit includes assessments of the overall architecture and design choices. Given the subjectivenature of these assessments, it will be up to the Verv energy development team to determine whether any changes should be made.

#### 2.2.2 Smart Contract Best Practices

This audit will evaluate whether the codebase follows the current established best practices for smartcontract development.

#### 2.2.3 Code Correctness

This audit will evaluate whether the code does what it is intended to do.

#### 2.2.4 Code Quality

This audit will evaluate whether the code has been written in a way that ensures readability and maintainability.

#### 2.2.5 Security

This audit will look for any exploitable security vulnerabilities, or other potential threats to either the operators of Verv or its users.

#### 2.2.6 Testing and testability

This audit will examine how easily tested the code is, and review how thoroughly tested the code is.

### 2.3 About the Verv Flux ICO

The specification of the ICO is specified in their README file.

After examining the source code, we verified that the audited source code behaves as follows.

* The Verv Flux ICO is an soft capped crowdsale.
* The public sale is set for four days between 31st March 1am BST and 4th April 1am BST though it is amendable.
* There is no minimum fundraising goal below which contributions are refunded.
* There is no hard cap encoded in the contract.
* The softcap can be increased until the crowdsale ends.
* There is no set token supply encoded in the contract.
* The inial tokne/ETH rate is 2000 and can be increased up to until the crowdsale starts.
* There are pre sale period where contract owner can allocate tokens either with variable rate (max8000)(disbursePreBuyersLkdContributions ) or for free ( distributePreBuyersLkdRewards).
* The presale allocated token must have vesting period of 3/6/9/12 months.
* All the preallocated tokens are held in the TokenVesting contract and can be withdrawn after the vesting periods.
* When Ether is sent, it is transfered to a wallet immediately 
(standard behavior of the CrowdSale contract).
* Day 1 is open to whitelisted users, with a 15% discount.
* Day 2 is open to whitelisted users, with a 12.5% discount.
* Day 3 is open to anyone, with a 10% discount.
* Day 4 has no discount.
* The crowdsale contract stops accepting contributions when either the cap or end time is reached.
* The sale is finalised when the finalize function is called by anyone.
* When the contract is finalised, it mints allocation for the company (34%).
* Once finalised no more tokens will be minted.

### 2.4 Terminology

This audit uses the following terminology.

#### 2.4.1 Likelihood

How likely a bug is to be encountered or exploited in the wild, as specified by the OWASP risk rating
methodology.

#### 2.4.2 Impact

The impact a bug would have if exploited, as specified by the OWASP risk rating methodology.

#### 2.4.3 Severity

How serious the issue is, derived from Likelihood and Impact as specified by the OWASP risk rating
methodology.

## Section 3 - Overview

### 3.1 Source Code

The Verv Flux smart contract source code was made available in the greenrunning/verv-smart-contract
Bitbucket repository.

The code was audited as of commit 24c98376861850773efcf26be63e45987ba76445 .

### 3.2 General Notes

The contract is built on top of OpenZeppelin smart contract libraries and custom stage transition,
white listing, and pre sale logics are added. The contract stage is transitioned via transitionGuard
modifier which are set on all public functions rather than calling a specific function.

The audited contracts uses OpenZeppelin v1.6.0 while the latest is v1.8.0 and some significant refactor-
ing on Crowdsale contract at v1.7.0. Though OpenZeppelin is one of the most popular smart contract
securities libraries, please be aware that their contract has not been publicly audited since March 2017.

### 3.3 Contracts

The following Solidity source files (with SHA1 sums) were audited:
* VervFluxCrowdsale.sol
* VervFluxToken.sol

MintableToken.sol , Pausable.sol , CappedCrowdsale.sol , and TokenVesting.sol are com-
mon library code and were NOT audited. We did not audit any code for the accounts that will be recipient
of crowdsale funds, or any code involved in subsequent allocation of tokens to users.

## Section 4 - Testing

There are automated tests and they follow the latest javascript syntax and styles. The test case covers
basic scenarios but no test coverage report are attached.

## Section 5 - Audit findings

### 5.1 Note Issues

#### 5.1.1 finalizationRetainStrategy[companyWallet] does not need to be a map
* Likelihood: low
* Impact: low
* Source

This can be simply a constant as the finalizationRetainStrategy map does not hold any key apart
from companyWallet and the value never gest updated.

#### 5.1.2 investorsMarge should not need to be calculated at finalize
* Likelihood: low
* Impact: low
* Source

The value does not depend on any of variables so can be just set as a constant.

#### 5.1.3 updateCap has no check for 20 mil hardcap which is mentioned at comment.
* Likelihood: low
* Impact: low
* Source

#### 5.1.4 wither cap is a typo
* Likelihood: low
* Impact: low
* Source

Rename to wether .

#### 5.1.5 bonuses variable defined but never used
* Likelihood: Low
* Impact: low
* Source

#### 5.1.6 whenNotPaused modifier is missing on disbursePreBuyersLkdContributions function
* Likelihood: Low
* Impact: low
* Source

whenNotPaused is set across all public functions except disbursePreBuyersLkdContributions. 
Even though it will be reverted by the same modifier at distributePreBuyersLkdRewards we
recommend adding it across all public functions to be consistent.

### 5.2 Low Issues

#### 5.2.1 Vesting period can be manipulated by changing endTime

* Likelihood: low
* Impact: medium
* Source 

At distributePreBuyersLkdRewards function, lockup period is set as endTime + PRESALE_LOCKUP_PERIOD. 
However, updateEndTime allows owner to change endTime during presale period. If the contract
owner is malicious, it can set endTime (and startTime ) 1 year earlier to netralise presale lockup
period. The simple solution is either:

A: Check transaction logs to make sure no such change have made. 

B: updateStartTime and updateEndTime should check if new date is older than now (assuming that the duration between the contract deployment and pre sale period are relatively short).

#### 5.2.2 updateCap does not check if new cap is more than weiRaised

* Likelihood: low
* Impact: medium
* Source

updateCap checks if new value is higher than existing cap but does not check if it is more than weiRaised . 
If the new cap is accidentally set lower than weiRaised the contract transition to Stages.SaleOver and the state is irreversible.

The recommendation is to add a check to make sure that new cap does not exceed weiRaised .

#### 5.2.3 start and end time are hardcoded on constructor

* Likelihood: low
* Impact: low
* Source

There are hard coded start and end times in the constructor, which has already passed. Even though
they can be changed by updateStartTime and updateEndTime , it is recommended that these
parameters are passed at constructor of the VervFluxCrowdsale contruct.

#### 5.2.4 hasEnded defined in super class and sub class which offer different functionality

* Likelihood: low
* Impact: low
* Source

There is confusion between function hasEnded in contract and same named function in super
( Crowdsale ), which tests different things. The function hasEnded in contract is not called directly
(but is public ). Recommend removing this function.

#### 5.2.5 uint for representing stage should be uint8
* Likelihood: low
* Impact: low
* Source = various places

There are multiple places where uint is used to represent stage enum which is only between 0~3.
We assume that uint was used to use less storage cost but uint is in fact just an alias to uint256
(Reference). We recommend changing the value to uint8 .

#### 5.2.6 Whitelisting may take too long

* Likelihood: high
* Impact: low
* Source

whitelistParticipant function only takes one address at a time. Depending on the network
congestion, it may take too long to add all the white list. Recommendation is to take an array of
addresses.

### 5.3 Medium Issues

None found.

### 5.4 High Issues

#### 5.4.1 Whitelisted users can contribute more than 10 ETH in day 1

* Likelihood: high
* Impact: medium
* Source

MAX_ALLOWED_FIRST_DAY_INVESTMENT is used to check on day 1 if msg.value has not exceeded
the limit of 10ETH. However it does not check if users has previously invested, hence it allows white
listed users to invest by calling buyTokens function with lower than 10 ETH multiple times.

### 5.4.2 Users can contribute more than cap

* Likelihood: high
* Impact: medium
* Source

In the OpenZeppelin libaray, cap check is done at validPurchase function which is only called via
buyTokens function but not via disbursePreBuyersLkdContributions .

```
function validPurchase() internal view returns (bool) {
bool withinCap = weiRaised.add(msg.value) <= cap;
return withinCap && super.validPurchase();
}

```

### 5.5 Critical Issues

None found.
