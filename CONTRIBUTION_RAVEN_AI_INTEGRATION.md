# Contribution Proposal: Raven AI + langchain-nvidia Integration

## Summary

This proposal outlines the integration of **Raven AI** (sovereign, local-first agentic platform for biology/healthcare) with **langchain-nvidia** (NVIDIA's LangChain integration for Nemotron, NIM, and NeMo models). The integration adds:

1. **NVIDIA Model Profiles** in Raven's provider routing system
2. **NVIDIAChatModelAdapter** for seamless Nemotron/NIM inference
3. **Token Economy Cost Tracking** for NVIDIA API pricing
4. **RAG Integration** with NV-Embed-QA and NV-Rerank-QA
5. **Clinical PHI-Aware Routing** for OpenClinical AI deployments
6. **Evidence Graph + Scientific Gates** for auditability

---

## Files Changed/Added in Raven AI

### 1. `runtime/provider_profiles.py` — Extended with NVIDIA Profiles

Added 10 new `ModelProviderProfile` entries:

| Profile ID | Model | Use Case |
|------------|-------|----------|
| `nvidia-nemotron-3-ultra` | Nemotron 3 Ultra (120B MoE) | Flagship agentic AI, strong remote lane |
| `nvidia-nemotron-3-super` | Nemotron 3 Super (53B MoE) | Cost-effective draft lane, tool-calling |
| `nvidia-nemotron-3-ultra-nim` | Self-hosted NIM | Sovereign/PHI deployments |
| `nvidia-nemotron-4-ultra` | Upcoming flagship | Future-proofing |
| `nvidia-nv-embed-qa` | NV-Embed-QA | RAG retrieval embeddings |
| `nvidia-nv-rerank-qa` | NV-Rerank-QA | Cross-encoder reranking |
| `nvidia-nim-local-chat` | Local NIM chat | Local-first chat |
| `nvidia-nim-local-embeddings` | Local NIM embeddings | Local RAG embeddings |

### 2. `runtime/models.py` — NVIDIAChatModelAdapter

```python
class NVIDIAChatModelAdapter(ModelAdapter):
    """Adapter for NVIDIA Nemotron models via langchain-nvidia-ai-endpoints."""
    
    def __init__(
        self,
        model: str = "nvidia/nemotron-3-super-53b-a8b",
        base_url: str = "https://integrate.api.nvidia.com/v1",
        api_key: str | None = None,
        system_prompt: str | None = None,
        thinking_mode: str = "non-thinking",
        max_tokens: int = 4096,
        temperature: float = 0.1,
    ) -> None:
        # Uses ChatNVIDIA from langchain_nvidia_ai_endpoints
        # Supports tool-calling, thinking mode, structured output
```

### 3. `runtime/token_economy.py` — NVIDIA Cost Integration

```python
NVIDIA_API_PRICING = {
    "nvidia/nemotron-3-ultra-120b-a12b": {"input_usd_per_1m": 5.00, "output_usd_per_1m": 15.00},
    "nvidia/nemotron-3-super-53b-a8b": {"input_usd_per_1m": 2.00, "output_usd_per_1m": 6.00},
    "nvidia/nemotron-3-ultra-120b-a12b-nim": {"input_usd_per_1m": 0.0, "output_usd_per_1m": 0.0},
    "nvidia/nv-embed-qa": {"input_usd_per_1m": 0.10, "output_usd_per_1m": 0.0},
    "nvidia/nv-rerank-qa": {"input_usd_per_1m": 1.00, "output_usd_per_1m": 0.0},
}

def estimate_nvidia_cost(model_id, input_tokens, output_tokens) -> dict:
    """Returns cost breakdown with GPT-5.5 / Opus 4.7 baselines for comparison."""
```

### 4. `runtime/provider_profiles.py` — NVIDIA-Aware Routing

```python
def select_provider(request: ProviderRouteRequest) -> ProviderRouteDecision:
    # PHI/private → Local-first
    # Public + tool-calling/JSON/latency-sensitive → Nemotron 3 Super (preferred)
    # High reasoning → Strong remote lane
    # Default → Cheap remote lane
```

### 5. `cookbook/raven_ai_nvidia_integration.ipynb` — Complete Example

A runnable Jupyter notebook demonstrating:
- Basic Nemotron chat (Ultra/Super)
- Token Economy cost planning
- RAG with NV-Embed-QA + NV-Rerank-QA
- Self-hosted NIM configuration
- Tool-calling agent with PubMed/DrugBank
- Evidence Graph trace generation
- Scientific Agent Gates evaluation
- Complete biology research workflow

### 6. `PAPER_NVIDIA_INTEGRATION.md` — Scientific Paper

Full technical documentation covering:
- Architecture & integration patterns
- Model registry & adapter implementation
- Token Economy with NVIDIA pricing
- RAG pipeline with NV-Embed/Rerank
- Clinical PHI-aware routing
- Evaluation results (BFCL, API-Bank, cost efficiency)
- Deployment guides (cloud API + self-hosted NIM)

---

## Proposed Contributions to langchain-nvidia

While the core integration lives in Raven AI, we propose the following enhancements to `langchain-nvidia` itself:

### 1. Add Raven AI Integration Example to Cookbook

**File:** `cookbook/raven_ai_integration.ipynb` (new)

A focused example showing:
- Basic Nemotron usage with `ChatNVIDIA`
- Nemotron 3 Ultra/Super for agentic workflows
- NV-Embed-QA + NV-Rerank-QA for RAG
- Local NIM deployment pattern

### 2. Enhanced Documentation for Nemotron Agentic Use Cases

**Files:** `libs/ai-endpoints/docs/chat/*.ipynb` (updates)

Add sections on:
- Tool-calling best practices with Nemotron
- Thinking mode activation patterns
- Structured output / JSON schema compliance
- Multi-step agent planning

### 3. NVIDIA Nemotron Model Card Updates

**Files:** `libs/ai-endpoints/docs/providers/nvidia.md`

Add:
- Agentic benchmark results (BFCL, API-Bank)
- Recommended configurations for tool-calling
- Thinking mode performance characteristics

---

## Testing

All integrations tested with:
- ✅ Unit tests for `provider_profiles.py` (profile selection, routing)
- ✅ Unit tests for `token_economy.py` (cost estimation, plan generation)
- ✅ Integration test: `NVIDIAChatModelAdapter` with mock `ChatNVIDIA`
- ✅ Cookbook notebook execution (requires `NVIDIA_API_KEY`)
- ✅ Scientific Agent Gates evaluation with Nemotron outputs

---

## Compatibility

| Dependency | Version |
|------------|---------|
| `langchain-nvidia-ai-endpoints` | >= 0.3.0 |
| `langchain-core` | >= 0.3.0 |
| `langgraph` | >= 0.2.0 (for parallel/speculative execution) |
| Python | >= 3.11 |

---

## Breaking Changes

**None.** All additions are backward-compatible:
- New provider profiles are additive
- New adapter is opt-in (requires explicit manifest `model_type: "nvidia-nemotron"`)
- Token Economy cost fields are optional with defaults
- Routing logic only affects NVIDIA-enabled deployments

---

## Migration Guide

For existing Raven AI deployments adding NVIDIA support:

```bash
# 1. Install NVIDIA integration
pip install langchain-nvidia-ai-endpoints

# 2. Set API key (for cloud) or deploy NIM (for local)
export NVIDIA_API_KEY="nvapi-..."

# 3. Update provider profiles (automatic via git pull)
# 4. Add NVIDIA model manifests to registry/
# 5. Restart Raven server
```

---

## Review Checklist

- [ ] Code style matches project conventions (ruff, mypy)
- [ ] All new code has type hints
- [ ] Documentation updated for new features
- [ ] Cookbook example runs end-to-end
- [ ] No breaking changes to public APIs
- [ ] Security review for PHI routing logic
- [ ] Cost estimation validated against NVIDIA API catalog

---

## Contact

**Author:** Barry Clerjuste (simpliibarrii@outlook.com)  
**Raven AI:** https://github.com/simpliibarrii-crypto/raven-ai  
**langchain-nvidia Fork:** https://github.com/simpliibarrii-crypto/langchain-nvidia  

**Discussion:** Open to feedback on:
- Pricing accuracy (NVIDIA API catalog may change)
- Additional Nemotron variants (Nemotron 4, Nemotron-Mini)
- Integration with `langchain-nvidia-langgraph` speculative execution
- OpenShell sandbox integration for tool execution

---

## License

All contributions follow the Apache-2.0 license of the respective repositories.