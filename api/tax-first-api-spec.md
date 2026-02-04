```

## 5. API Layer Rewritten

### `rickcreator87-credit-tools/api/tax-first-api-spec.md`
```markdown
# Tax-First API Specification
## RickCreator87 Credit Authority System

## API VERSION: 2.0.0
## BASE URL: https://api.rickcreator87-credit-authority.com/v2

## AUTHENTICATION
All endpoints require JWT authentication with dual-founder permissions.

## 1. TAX BLOCK API

### 1.1 Generate Federal Tax Block
```http
POST /api/v2/tax/federal/generate
Authorization: Bearer <dual_founder_jwt>
Content-Type: application/json

Request:
{
  "transaction_id": "TRX_[UUID]",
  "borrower_tax_id": "[TAX_ID]",
  "transaction_amount": 10000.00,
  "transaction_type": "personal_loan",
  "estimated_income": 50000.00,
  "withholding_preferences": {
    "federal_rate": 0.22,
    "additional_withholding": 0.0
  }
}

Response:
{
  "status": "success",
  "tax_block": {
    "federal_tax_block_id": "FTB_2024_[UUID]",
    "generation_date": "2024-01-15T10:30:00Z",
    "valid_until": "2024-12-31T23:59:59Z",
    "tax_liability_amount": 2200.00,
    "withholding_requirements": [
      {
        "type": "federal_income_tax",
        "rate": 0.22,
        "amount": 2200.00,
        "withholding_account": "FED_WH_[UUID]",
        "due_date": "2024-04-15"
      }
    ],
    "compliance_status": "PENDING_APPROVAL",
    "approval_workflow": {
      "requires_compliance_officer": true,
      "requires_dual_founder": true,
      "current_step": "generation_complete"
    }
  },
  "audit_trail": {
    "generation_hash": "[SHA256]",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

1.2 Generate State Tax Block

```http
POST /api/v2/tax/state/generate
Authorization: Bearer <dual_founder_jwt>
Content-Type: application/json

Request:
{
  "transaction_id": "TRX_[UUID]",
  "federal_tax_block_id": "FTB_2024_[UUID]",
  "state_code": "CA",
  "state_tax_id": "[STATE_TAX_ID]",
  "transaction_amount": 10000.00,
  "state_withholding_preferences": {
    "state_rate": 0.093,
    "local_tax": true
  }
}

Response:
{
  "status": "success",
  "tax_block": {
    "state_tax_block_id": "STB_CA_2024_[UUID]",
    "state_code": "CA",
    "generation_date": "2024-01-15T10:35:00Z",
    "valid_until": "2024-12-31T23:59:59Z",
    "state_tax_liability": 930.00,
    "state_withholding": [
      {
        "type": "state_income_tax",
        "rate": 0.093,
        "amount": 930.00,
        "withholding_account": "CA_WH_[UUID]",
        "due_date": "2024-04-15"
      }
    ],
    "compliance_status": "PENDING_APPROVAL",
    "approval_workflow": {
      "requires_compliance_officer": true,
      "requires_dual_founder": true,
      "current_step": "generation_complete"
    }
  },
  "audit_trail": {
    "generation_hash": "[SHA256]",
    "timestamp": "2024-01-15T10:35:00Z"
  }
}
```

1.3 Approve Tax Blocks

```http
POST /api/v2/tax/blocks/approve
Authorization: Bearer <compliance_jwt>
Content-Type: application/json

Request:
{
  "tax_block_ids": ["FTB_2024_[UUID]", "STB_CA_2024_[UUID]"],
  "approver_role": "compliance_officer",
  "approver_signature_hash": "[SIGNATURE_HASH]",
  "approval_notes": "Tax liability calculations verified"
}

Response:
{
  "status": "success",
  "approved_blocks": {
    "federal_tax_block_id": "FTB_2024_[UUID]",
    "state_tax_block_id": "STB_CA_2024_[UUID]",
    "approval_status": "COMPLIANCE_APPROVED",
    "approval_timestamp": "2024-01-15T11:00:00Z",
    "next_approval_step": "dual_founder_approval"
  },
  "workflow_progress": {
    "current_milestone": "tax_block_approval",
    "completed_steps": ["generation", "compliance_approval"],
    "pending_steps": ["dual_founder_approval"]
  }
}
```

2. AGREEMENT API

2.1 Create Agreement from Template

```http
POST /api/v2/agreements/create
Authorization: Bearer <dual_founder_jwt>
Content-Type: application/json

Request:
{
  "template_type": "personal_loan_agreement",
  "tax_block_ids": ["FTB_2024_[UUID]", "STB_CA_2024_[UUID]"],
  "agreement_data": {
    "borrower_name": "John Doe",
    "principal_amount": 10000.00,
    "interest_rate": 0.05,
    "term_months": 24,
    "repayment_schedule": "monthly",
    "collateral": "none"
  },
  "workflow_version": "2.0.0"
}

Response:
{
  "status": "success",
  "agreement": {
    "agreement_id": "AGR_[UUID]",
    "template_type": "personal_loan_agreement",
    "generated_date": "2024-01-15T11:30:00Z",
    "tax_block_references": ["FTB_2024_[UUID]", "STB_CA_2024_[UUID]"],
    "agreement_status": "DRAFT",
    "signatures_required": 2,
    "execution_workflow": {
      "step_1": "tax_block_validation",
      "step_2": "dual_founder_review",
      "step_3": "signature_collection",
      "step_4": "ledger_certification"
    }
  },
  "validation": {
    "tax_blocks_valid": true,
    "template_valid": true,
    "data_complete": true
  }
}
```

2.2 Execute Agreement with Signatures

```http
POST /api/v2/agreements/execute
Authorization: Bearer <dual_founder_jwt>
Content-Type: application/json

Request:
{
  "agreement_id": "AGR_[UUID]",
  "signatures": [
    {
      "founder": "RickCreator87",
      "signature_hash": "[SIGNATURE_HASH_1]",
      "signing_timestamp": "2024-01-15T12:00:00Z",
      "approval_type": "tax_first_approval"
    },
    {
      "founder": "[CO_FOUNDER_NAME]",
      "signature_hash": "[SIGNATURE_HASH_2]",
      "signing_timestamp": "2024-01-15T12:05:00Z",
      "approval_type": "tax_first_approval"
    }
  ],
  "tax_block_validation": {
    "revalidate_blocks": true,
    "compliance_status_check": true
  }
}

Response:
{
  "status": "success",
  "execution_result": {
    "agreement_id": "AGR_[UUID]",
    "execution_date": "2024-01-15T12:05:00Z",
    "execution_status": "EXECUTED",
    "signatures_collected": 2,
    "tax_compliance_verified": true,
    "ledger_certificate_required": true,
    "next_milestone": "ledger_entry"
  },
  "certificates": {
    "execution_certificate_hash": "[SHA256]",
    "tax_compliance_certificate": "TAX_CERT_[UUID]"
  }
}
```

3. LEDGER API

3.1 Create Ledger Entry

```http
POST /api/v2/ledger/entries
Authorization: Bearer <dual_founder_jwt>
Content-Type: application/json

Request:
{
  "ledger_type": "personal", // or "organizational"
  "agreement_id": "AGR_[UUID]",
  "tax_block_ids": ["FTB_2024_[UUID]", "STB_CA_2024_[UUID]"],
  "financial_data": {
    "gross_amount": 10000.00,
    "tax_withheld": 3130.00,
    "net_amount": 6870.00,
    "currency": "USD",
    "transaction_date": "2024-01-15"
  },
  "approval_data": {
    "founder_1_approval": {
      "signature_hash": "[SIGNATURE_HASH_1]",
      "approval_timestamp": "2024-01-15T12:10:00Z"
    },
    "founder_2_approval": {
      "signature_hash": "[SIGNATURE_HASH_2]",
      "approval_timestamp": "2024-01-15T12:15:00Z"
    }
  },
  "workflow_version": "2.0.0"
}

Response:
{
  "status": "success",
  "ledger_entry": {
    "entry_id": "PLE_[UUID]",
    "ledger_type": "personal",
    "creation_date": "2024-01-15T12:20:00Z",
    "tax_separation_flag": true,
    "federal_tax_block_id": "FTB_2024_[UUID]",
    "state_tax_block_id": "STB_CA_2024_[UUID]",
    "workflow_version": "2.0.0",
    "dual_founder_approval": {
      "founder_1_approval": {
        "status": "APPROVED",
        "timestamp": "2024-01-15T12:10:00Z"
      },
      "founder_2_approval": {
        "status": "APPROVED",
        "timestamp": "2024-01-15T12:15:00Z"
      }
    },
    "audit_hash": "[SHA256_HASH]",
    "ledger_certificate_id": "LCERT_[UUID]"
  },
  "validation": {
    "tax_blocks_valid": true,
    "agreement_valid": true,
    "approvals_valid": true,
    "amounts_reconciled": true
  }
}
```

3.2 Generate Ledger Certificate

```http
POST /api/v2/ledger/certificates
Authorization: Bearer <system_jwt>
Content-Type: application/json

Request:
{
  "ledger_entry_ids": ["PLE_[UUID]", "OLE_[UUID]"],
  "agreement_id": "AGR_[UUID]",
  "tax_block_ids": ["FTB_2024_[UUID]", "STB_CA_2024_[UUID]"],
  "certification_type": "dual_ledger_certification"
}

Response:
{
  "status": "success",
  "certificate": {
    "ledger_certificate_id": "LCERT_[UUID]",
    "certification_date": "2024-01-15T12:25:00Z",
    "certified_entries": ["PLE_[UUID]", "OLE_[UUID]"],
    "tax_separation_verified": true,
    "dual_founder_approval_verified": true,
    "workflow_compliance": "2.0.0",
    "audit_hash": "[SHA256_HASH]",
    "validation_checks": {
      "tax_blocks_linked": true,
      "agreement_referenced": true,
      "amounts_matching": true,
      "approvals_present": true
    }
  }
}
```

4. DISBURSEMENT API

4.1 Initiate Disbursement

```http
POST /api/v2/disbursements/initiate
Authorization: Bearer <dual_founder_jwt>
Content-Type: application/json

Request:
{
  "agreement_id": "AGR_[UUID]",
  "ledger_certificate_id": "LCERT_[UUID]",
  "disbursement_data": {
    "recipient_name": "John Doe",
    "recipient_account": "[ACCOUNT_INFO]",
    "disbursement_amount": 6870.00,
    "disbursement_method": "wire_transfer",
    "scheduled_date": "2024-01-16"
  },
  "tax_validation": {
    "revalidate_tax_blocks": true,
    "verify_withholding": true
  }
}

Response:
{
  "status": "success",
  "disbursement": {
    "disbursement_id": "DISB_[UUID]",
    "initiation_date": "2024-01-15T12:30:00Z",
    "agreement_reference": "AGR_[UUID]",
    "ledger_certificate_reference": "LCERT_[UUID]",
    "tax_blocks_validated": ["FTB_2024_[UUID]", "STB_CA_2024_[UUID]"],
    "disbursement_status": "PENDING_APPROVAL",
    "approval_workflow": {
      "requires_compliance": true,
      "requires_dual_founder": true,
      "current_step": "initiation_complete"
    },
    "validation_gates": {
      "tax_validation_gate": "PASSED",
      "agreement_validation_gate": "PASSED",
      "ledger_validation_gate": "PASSED",
      "approval_validation_gate": "PENDING"
    }
  }
}
```

4.2 Approve Disbursement

```http
POST /api/v2/disbursements/approve
Authorization: Bearer <dual_founder_jwt>
Content-Type: application/json

Request:
{
  "disbursement_id": "DISB_[UUID]",
  "approver_role": "dual_founder",
  "approver_signature_hash": "[SIGNATURE_HASH]",
  "final_validation": {
    "tax_blocks_current": true,
    "amounts_correct": true,
    "recipient_verified": true
  }
}

Response:
{
  "status": "success",
  "disbursement_approval": {
    "disbursement_id": "DISB_[UUID]",
    "approval_status": "APPROVED",
    "approval_timestamp": "2024-01-15T13:00:00Z",
    "approved_by": ["RickCreator87", "[CO_FOUNDER_NAME]"],
    "final_validation": {
      "tax_compliance": "VERIFIED",
      "agreement_compliance": "VERIFIED",
      "ledger_compliance": "VERIFIED",
      "workflow_compliance": "2.0.0"
    },
    "execution_ready": true
  }
}
```

4.3 Execute Disbursement

```http
POST /api/v2/disbursements/execute
Authorization: Bearer <system_jwt>
Content-Type: application/json

Request:
{
  "disbursement_id": "DISB_[UUID]",
  "execution_timestamp": "2024-01-16T09:00:00Z",
  "execution_method": "automated_wire",
  "transaction_reference": "[BANK_REF]"
}

Response:
{
  "status": "success",
  "disbursement_execution": {
    "disbursement_id": "DISB_[UUID]",
    "execution_date": "2024-01-16T09:00:00Z",
    "execution_status": "COMPLETED",
    "transaction_reference": "[BANK_REF]",
    "amount_disbursed": 6870.00,
    "tax_withheld": 3130.00,
    "total_transaction": 10000.00,
    "final_audit_hash": "[SHA256_HASH]",
    "workflow_completion": {
      "milestone": "disbursement_complete",
      "all_gates_passed": true,
      "tax_first_enforced": true,
      "audit_trail_complete": true
    }
  }
}
```

5. AUDIT API

5.1 Retrieve Audit Trail

```http
GET /api/v2/audit/trail/{entity_id}
Authorization: Bearer <dual_founder_jwt>
```

Response:

```json
{
  "status": "success",
  "audit_trail": {
    "entity_id": "AGR_[UUID]",
    "trail_type": "agreement_lifecycle",
    "entries": [
      {
        "timestamp": "2024-01-15T10:30:00Z",
        "action": "federal_tax_block_generated",
        "actor": "system",
        "details": {
          "tax_block_id": "FTB_2024_[UUID]",
          "amount": 2200.00
        },
        "hash": "[SHA256_HASH_1]"
      },
      {
        "timestamp": "2024-01-15T10:35:00Z",
        "action": "state_tax_block_generated",
        "actor": "system",
        "details": {
          "tax_block_id": "STB_CA_2024_[UUID]",
          "amount": 930.00
        },
        "hash": "[SHA256_HASH_2]"
      },
      {
        "timestamp": "2024-01-15T11:00:00Z",
        "action": "tax_blocks_approved",
        "actor": "compliance_officer",
        "details": {
          "approver": "[OFFICER_NAME]",
          "status": "COMPLIANCE_APPROVED"
        },
        "hash": "[SHA256_HASH_3]"
      },
      {
        "timestamp": "2024-01-15T11:30:00Z",
        "action": "agreement_created",
        "actor": "RickCreator87",
        "details": {
          "agreement_id": "AGR_[UUID]",
          "template": "personal_loan_agreement"
        },
        "hash": "[SHA256_HASH_4]"
      },
      {
        "timestamp": "2024-01-15T12:00:00Z",
        "action": "agreement_signed",
        "actor": "RickCreator87",
        "details": {
          "signature_type": "founder_1"
        },
        "hash": "[SHA256_HASH_5]"
      },
      {
        "timestamp": "2024-01-15T12:05:00Z",
        "action": "agreement_signed",
        "actor": "[CO_FOUNDER_NAME]",
        "details": {
          "signature_type": "founder_2"
        },
        "hash": "[SHA256_HASH_6]"
      },
      {
        "timestamp": "2024-01-15T12:20:00Z",
        "action": "ledger_entry_created",
        "actor": "system",
        "details": {
          "entry_id": "PLE_[UUID]",
          "ledger_type": "personal"
        },
        "hash": "[SHA256_HASH_7]"
      },
      {
        "timestamp": "2024-01-15T12:25:00Z",
        "action": "ledger_certificate_generated",
        "actor": "system",
        "details": {
          "certificate_id": "LCERT_[UUID]"
        },
        "hash": "[SHA256_HASH_8]"
      },
      {
        "timestamp": "2024-01-15T12:30:00Z",
        "action": "disbursement_initiated",
        "actor": "RickCreator87",
        "details": {
          "disbursement_id": "DISB_[UUID]",
          "amount": 6870.00
        },
        "hash": "[SHA256_HASH_9]"
      },
      {
        "timestamp": "2024-01-15T13:00:00Z",
        "action": "disbursement_approved",
        "actor": "dual_founder",
        "details": {
          "approvers": ["RickCreator87", "[CO_FOUNDER_NAME]"]
        },
        "hash": "[SHA256_HASH_10]"
      },
      {
        "timestamp": "2024-01-16T09:00:00Z",
        "action": "disbursement_executed",
        "actor": "system",
        "details": {
          "transaction_reference": "[BANK_REF]",
          "status": "COMPLETED"
        },
        "hash": "[SHA256_HASH_11]"
      }
    ],
    "trail_hash": "[FINAL_SHA256_HASH]",
    "validation": {
      "chain_integrity": true,
      "no_gaps": true,
      "signatures_valid": true,
      "tax_first_compliant": true
    }
  }
}
```

6. VALIDATION API

6.1 Validate Tax-First Compliance

```http
POST /api/v2/validate/compliance
Authorization: Bearer <compliance_jwt>
Content-Type: application/json

Request:
{
  "entity_id": "AGR_[UUID]",
  "entity_type": "agreement",
  "validation_type": "tax_first_compliance",
  "validation_rules": ["2.0.0"]
}

Response:
{
  "status": "success",
  "validation_result": {
    "entity_id": "AGR_[UUID]",
    "validation_type": "tax_first_compliance",
    "rules_version": "2.0.0",
    "checks_performed": [
      {
        "check": "tax_blocks_present",
        "status": "PASSED",
        "details": {
          "federal_tax_block": "FTB_2024_[UUID]",
          "state_tax_block": "STB_CA_2024_[UUID]"
        }
      },
      {
        "check": "tax_blocks_approved",
        "status": "PASSED",
        "details": {
          "compliance_approval": true,
          "dual_founder_approval": true
        }
      },
      {
        "check": "tax_first_gating",
        "status": "PASSED",
        "details": {
          "agreement_gated_by_tax": true,
          "ledger_gated_by_tax": true,
          "disbursement_gated_by_tax": true
        }
      },
      {
        "check": "dual_founder_approval",
        "status": "PASSED",
        "details": {
          "founder_1_approval": true,
          "founder_2_approval": true,
          "approval_order": "tax_first"
        }
      },
      {
        "check": "workflow_compliance",
        "status": "PASSED",
        "details": {
          "milestones_in_order": true,
          "gates_respected": true,
          "version_compliance": "2.0.0"
        }
      }
    ],
    "overall_compliance": "COMPLIANT",
    "certificate": "COMPLIANCE_CERT_[UUID]",
    "audit_reference": "[AUDIT_HASH]"
  }
}
```

ERROR HANDLING

All endpoints return standard error responses:

```json
{
  "status": "error",
  "error_code": "TAX_BLOCK_INVALID",
  "message": "Federal tax block validation failed",
  "details": {
    "tax_block_id": "FTB_2024_[UUID]",
    "validation_errors": ["Block expired", "Approval missing"],
    "suggested_action": "Regenerate tax block with current rates"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

RATE LIMITING

· Compliance endpoints: 10 requests/minute
· Founder endpoints: 100 requests/minute
· System endpoints: 1000 requests/minute
· Audit endpoints: 50 requests/minute
