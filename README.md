# DAM Transfer Fee ERC20 Audit Report - Mudit Gupta

## Objective of the audit

The audit's focus is to verify that the smart contract system is secure, resilient, and working according to its specifications. The audit activities can be grouped in the following three categories:

**Security**: Identifying security related issues within each contract and the system of contracts.

**Sound Architecture**: Evaluation of this system's architecture through the lens of established smart contract best practices and general software best practices.

**Code Correctness and Quality**: A full audit of the contract source code.

This audit is based on commit hash `bb734f8a09f8bb5ce10824f57ee7012be42a4356` of <https://github.com/DecentralizedAssetManagement/SetV2Customization>. The fixes were verified on commit hash `1c7c3e36e22c7ec3c1b01859752e7cb8ede2fd16`.

## Findings

During the audit, 0 Critical, 1 Major, 2 Minor, and 6 Informational issues were found.

- A critical issue represents something that can be relatively easily exploited and will likely lead to loss of funds.
- A major issue represents something that can result in an unintended behavior of the smart contracts. These issues can also lead to loss of funds but are typically harder to execute than critical issues.
- A minor issue represents an oddity discovered during the audit. These issues are typically situational or hard to exploit.
- An informational issue represents a potential improvement. These issues do not pose any practical security risks.

### Major

##### 1.1 Wrapper Tokens can be used to bypass transfer fees

##### Description

The exit/transfer fee can be bypassed by using a wrapper token. The wrapper token will work like the WETH token where every wrapped token will be backed by the underlying token. Users will be able to wrap/unwrap at any time. The wrapped token can be traded freely without paying any transfer fees. To dissuade creation of wrapper tokens, It's recommend to add a blacklist feature that can be used to trap the collateral/tokens held by the wrapper token.

##### Status update

The blacklist features has been added.

### Minor

#### 2.1 Uniswap Liquidity Tokens can be used to bypass transfer fees.

##### Description

Adding and withdrawing Liquidity from Uniswap will be exempt from paying transfer fees. Users can add liquidity, freely trade the LP tokens and then withdraw the liquidity using those LP tokens to do a fee-less transfer essentially. This method has little practical use though since the sender will have to risk equal amount of ETH to add Liquidity and trust that the receiver returns the ETH. This also involves additional gas costs so it will be cheaper to just pay the transfer fee in small-medium trades. Considering everything, this seems like an acceptable risk but something to keep in mind regardless.

#### 2.2 Unnecessary storage operations.

##### Description

Calling `_transfer` twice in transfer functions is expensive since the sender's balance will be updated twice. Consider adding a custom `_transfer` function that updates the sender's balance only once. If you modify the `_transfer` function, you won't need to modify `transfer`/`transferFrom` at all.

##### Status update

This issue has been fixed using a custom `_transfer` function.

### Informational

**NOTE**: All of these suggestions have been implemented or made moot by other changes already.

#### 3.1 Code simplification

The following snippet can be simplified

```solidity
if (feeExemptFromAddresses[sender] || feeExemptToAddresses[recipient]) {
    return true;
} else {
    return false;
}
```

to

```solidity
return (feeExemptFromAddresses[sender] || feeExemptToAddresses[recipient]);
```

#### 3.2 Extra long reason strings

Reason strings are part of the onchain code and are stored in chunks of 32 bytes. It's recommended to keep them short (within 32 bytes) to avoid paying extra fees. Long string like "Only the current manager can take privileged actions (e.g. adding addresses exempt from the fee)" should be avoided.

#### 3.3 Incorrect reason string

In `uint256 amountAfterFee = amount.sub(feeAmount, "ERC20: transfer amount exceeds balance");`, The reason string does not make sense. Balance is not being checked here. The check is fine, just the error message is incorrect.

#### 3.4 Unused variables

`Pooltoken` and `liquidity` variables are unused, they should be removed.

#### 3.5 Code duplication

L53, L60 of TransferFeeERC20 `_approve(sender, msg.sender, allowance(sender, msg.sender).sub(amount, "ERC20: transfer amount exceeds allowance (no fee)"));` can be deduped by moving iy outside the if/else. Remaining code in transferFrom can be deduped with the code in transfer function

#### 3.6 Gas savings using constants

Since `dexRouter` can not be changed, it should be declared as a constant to save gas. Also consider making the `token` in `addLiquidityETH` function a constant since it is supposed to be the same contract always.

## Static analysis

Slither was used to do static analysis of the contracts and the output of slither can be found at <https://gist.github.com/maxsam4/a82d4151ebfa6d2ae41e6ea05aab9168>

**NOTE**: Please keep in mind that most of the issues flagged by Slither are false positives. The slither output was processed manually and incorporated with my other findings.

## Disclaimer

This report is not an endorsement or indictment of any particular project or team, and the report do not guarantee the security of any particular project. This report does not consider, and should not be interpreted as considering or having any bearing on, the potential economics of a token, token sale or any other product, service or other asset. Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty. This report does not provide any warranty or representation to any Third-Party in any respect, including regarding the bugfree nature of code, the business model or proprietors of any such business model, and the legal compliance of any such business. No third party should rely on this report in any way, including for the purpose of making any decisions to buy or sell any token, product, service or other asset. Specifically, for the avoidance of doubt, this report does not constitute investment advice, is not intended to be relied upon as investment advice, is not an endorsement of this project or team, and it is not a guarantee as to the absolute security of the project. I owe no duty to any Third-Party by virtue of publishing these Reports.

The scope of my audit is limited to a audit of Solidity code and only the Solidity code noted as being within the scope of the audit within this report. The Solidity language itself remains under development and is subject to unknown risks and flaws. The audit does not extend to the compiler layer, or any other areas beyond Solidity that could present security risks. Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty.

This audit does not give any warranties on finding all possible security issues of the given smart contracts, i.e., the evaluation result does not guarantee the nonexistence of any further findings of security issues. As one audit cannot be considered comprehensive, I always recommend proceeding with several independent audits and a public bug bounty program to ensure the security of smart contracts.
