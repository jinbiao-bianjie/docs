---
id: weave-transaction-spec
title: Build a transaction
sidebar_label: Build a transaction
---

IOV Name Service uses protobuf for handling communications.

A good starting point would be to check out the _proto_ files and become familiar with what they are and how they look.

## Protocol buffers

Let's start with explaining what protobufs are and how to use them:

> Protocol Buffers (a.k.a., protobuf) are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data.

This line is from [protobuf github repo](https://github.com/protocolbuffers/protobuf)

In a nutshell, protobuf is an effective and easy-to-use serialization format. To use it define protoc definitions(_.proto_) and then compile them to the desired language which protocol buffers [supports](https://developers.google.com/protocol-buffers/docs/tutorials). After these steps, high tech comms is ready to be used in your software.

### IOV Name Service proto definitions

_IOV Name Service_ protobuf definitions are kept under [weave/spec/proto](https://github.com/iov-one/weave/tree/v0.21.0/spec/proto) for easy access and usage. Dive in to _weave/spec/proto_ to get some idea of how _IOV Name Service codec_ files are defined and structured.

## IOV Name Service Transactions

Lets start explaining [tx codec](https://github.com/iov-one/weave/blob/master/spec/proto/cmd/bnsd/app/codec.proto).

All the required _proto_ files for connecting to `BNS` or any `Weave` based blockchain is using this `tx codec`.

On the first line of the `codec.proto`

```protobuf
syntax = "proto3";
```

This line indicates the protobuf file uses `proto3` as language syntax,

Next we define the package to which this codec belongs.

```protobuf
package bnsd;
```

Every codec message that will be used to interact with `bnsd` is boiled down to this codec file via _imports_

```protobuf
import "cmd/bnsd/x/account/codec.proto";
import "cmd/bnsd/x/preregistration/codec.proto";
import "cmd/bnsd/x/qualityscore/codec.proto";
import "cmd/bnsd/x/termdeposit/codec.proto";
import "cmd/bnsd/x/username/codec.proto";
import "datamigration/codec.proto";
import "migration/codec.proto";
import "x/aswap/codec.proto";
import "x/cash/codec.proto";
import "x/currency/codec.proto";
import "x/distribution/codec.proto";
import "x/escrow/codec.proto";
import "x/gov/codec.proto";
import "x/msgfee/codec.proto";
import "x/multisig/codec.proto";
import "x/sigs/codec.proto";
import "x/txfee/codec.proto";
import "x/validators/codec.proto";
```

### Tx message

```protobuf
message Tx {
  cash.FeeInfo fees = 1;
  repeated sigs.StdSignature signatures = 2;
  // Multisig contains IDs of multisig contracts.
  // empty array results in multisig being ignored, basically the same behaviour
  // as if it never got supplied.
  repeated bytes multisig = 4;
  // msg is a sum type over all allowed messages on this chain.
  oneof sum {
    cash.SendMsg cash_send_msg = 51;
    escrow.CreateMsg escrow_create_msg = 52;
    escrow.ReleaseMsg escrow_release_msg = 53;
    escrow.ReturnMsg escrow_return_msg = 54;
    escrow.UpdatePartiesMsg escrow_update_parties_msg = 55;
    multisig.CreateMsg multisig_create_msg = 56;
    multisig.UpdateMsg multisig_update_msg = 57;
    validators.ApplyDiffMsg validators_apply_diff_msg = 58;
    currency.CreateMsg currency_create_msg = 59;
    ExecuteBatchMsg execute_batch_msg = 60;
    username.RegisterTokenMsg username_register_token_msg = 61;
    username.TransferTokenMsg username_transfer_token_msg = 62;
    username.ChangeTokenTargetsMsg username_change_token_targets_msg = 63;
    username.UpdateConfigurationMsg username_update_configuration_msg = 64;
    distribution.CreateMsg distribution_create_msg = 66;
    distribution.DistributeMsg distribution_msg = 67;
    distribution.ResetMsg distribution_reset_msg = 68;
    migration.UpgradeSchemaMsg migration_upgrade_schema_msg = 69;
    aswap.CreateMsg aswap_create_msg = 70;
    aswap.ReleaseMsg aswap_release_msg = 71;
    aswap.ReturnMsg aswap_return_msg = 72;
    gov.CreateProposalMsg gov_create_proposal_msg = 73;
    gov.DeleteProposalMsg gov_delete_proposal_msg = 74;
    gov.VoteMsg gov_vote_msg = 75;
    // Tally is executed via cron only.
    // gov.TallyMsg gov_tally_msg = 76;
    gov.UpdateElectorateMsg gov_update_electorate_msg = 77;
    gov.UpdateElectionRuleMsg gov_update_election_rule_msg = 78;
    // 79 is reserved (see ProposalOptions: TextResolutionMsg)
    msgfee.SetMsgFeeMsg msgfee_set_msg_fee_msg = 80;
    datamigration.ExecuteMigrationMsg datamigration_execute_migration_msg = 81;
    account.UpdateConfigurationMsg account_update_configuration_msg = 82;
    account.RegisterDomainMsg account_register_domain_msg = 83;
    account.ReplaceAccountMsgFeesMsg account_replace_account_msg_fees_msg = 84;
    account.TransferDomainMsg account_transfer_domain_msg = 85;
    account.RenewDomainMsg account_renew_domain_msg = 86;
    account.DeleteDomainMsg account_delete_domain_msg = 87;
    account.RegisterAccountMsg account_register_account_msg = 88;
    account.TransferAccountMsg account_transfer_account_msg = 89;
    account.ReplaceAccountTargetsMsg account_replace_account_targets_msg = 90;
    account.DeleteAccountMsg account_delete_account_msg = 91;
    account.FlushDomainMsg account_flush_domain_msg = 92;
    account.RenewAccountMsg account_renew_account_msg = 93;
    account.AddAccountCertificateMsg account_add_account_certificate_msg = 94;
    account.DeleteAccountCertificateMsg account_delete_account_certificate_msg = 95;
    preregistration.RegisterMsg preregistration_register_msg = 96;
    cash.UpdateConfigurationMsg cash_update_configuration_msg = 97;
    txfee.UpdateConfigurationMsg txfee_update_configuration_msg = 98;
    termdeposit.CreateDepositContractMsg termdeposit_create_deposit_contract_msg = 99;
    termdeposit.DepositMsg termdeposit_deposit_msg = 100;
    termdeposit.ReleaseDepositMsg termdeposit_release_deposit_msg = 101;
    termdeposit.UpdateConfigurationMsg termdeposit_update_configuration_msg = 102;
    qualityscore.UpdateConfigurationMsg qualityscore_update_configuration_msg = 103;
    preregistration.UpdateConfigurationMsg preregistration_update_configuration_msg = 104;
    msgfee.UpdateConfigurationMsg msgfee_update_configuration_msg = 105;
  }
}
```

`oneof sum` in the _tx message_ means, one of the messages will be sent to _bnsd_. _reference:_ [developers.google.com/protocol-buffers](https://developers.google.com/protocol-buffers/docs/proto3#oneof)

## Communicating with the IOV Name Service

To import the required tools to communicate and send transactions to _IOV Name Service_, follow these steps.

### 1 - Import _bnsd_ protobuf definitions

- Copy the entire [weave/spec/proto/](https://github.com/iov-one/weave/tree/master/spec/proto) into the repo, as the imports should refer to local files.

### 2 - Install your preferred language-specific protoc plugin

> We recommend using [uber/prototool](https://github.com/uber/prototool) or better [IOV/prototool-docker](https://github.com/iov-one/prototool-docker). Import _prototool-docker_ directly from [docker hub](https://hub.docker.com/r/iov1/prototool)

- Create _prototool.yaml_ under root path.
- reference: [weave-starter-kit](https://github.com/iov-one/weave-starter-kit/blob/master/prototool.yaml)
- Import preferred languages plugin under _plugins_ with _plugin options_ such as compilation output.
- reference: [weave-starter-kit](https://github.com/iov-one/weave-starter-kit/blob/master/prototool.yaml#L22...L25)

### 3 - Import _app/codec.proto_

- Insert _app/codec.proto_ files path under `includes` in prototool.yaml
- reference: [weave-starter-kit](https://github.com/iov-one/weave-starter-kit/blob/master/prototool.yaml#L20)

### 4 - Compile _.proto_ definitions

- If _IOV/prototool_ used, run `docker run --rm --user=$(shell id -u):$(shell id -g) -v $(shell pwd):/work iov1/prototool:v0.2.2 prototool generate`
- Or instead of messing around with long protoc build scripts, include and parameterize this script in a build tool or script such as [weave-starter-kit/Makefile](https://github.com/iov-one/weave-starter-kit/blob/master/Makefile)

### 5 - Import compiled files in your project

- After all these steps, now the _weave/bnsd_ compiled protobuf files are ready. Import and make your dreams come true.
