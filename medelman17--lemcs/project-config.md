---
trigger: always_on
description: FastAPI development guidelines for legal document processing endpoints
---


# FastAPI Development Guidelines

## API Architecture

The LeMCS API provides endpoints for legal document processing, analysis, and consolidation using FastAPI with async patterns.

### Core API Structure
```
api/
├── routes/
│   ├── documents.py          # Document upload and processing
│   ├── citations.py          # Citation extraction endpoints
│   ├── consolidation.py      # Document consolidation
│   ├── health.py            # Health check endpoints
│   └── documents_simple.py  # Simplified document endpoints
```

## Endpoint Development Patterns

### Document Processing Endpoint
```python
from fastapi import APIRouter, UploadFile, File, Depends, HTTPException
from nlp.hybrid_legal_nlp import HybridLegalNLP
from typing import Dict, Any
import asyncio

router = APIRouter(prefix="/documents", tags=["documents"])

@router.post("/analyze")
async def analyze_legal_document(
    document: UploadFile = File(...),
    nlp_service: HybridLegalNLP = Depends(get_hybrid_nlp)
) -> Dict[str, Any]:
    """
    Analyze uploaded legal document with comprehensive NLP processing.
    
    Returns:
        - Document type classification
        - Entity extraction results
        - Legal concept analysis
        - Confidence scores
    """
    try:
        # Validate file type
        if not document.content_type.startswith('text/'):
            raise HTTPException(
                status_code=400, 
                detail="Only text files are supported"
            )
        
        # Extract text content
        content = await document.read()
        text = content.decode('utf-8')
        
        # Process with legal NLP
        loop = asyncio.get_event_loop()
        analysis = await loop.run_in_executor(
            None, nlp_service.comprehensive_analysis, text
        )
        
        return {
            "status": "success",
            "filename": document.filename,
            "document_type": analysis["document_analysis"]["type"],
            "confidence": analysis["document_analysis"]["confidence"],
            "analysis": analysis
        }
        
    except UnicodeDecodeError:
        raise HTTPException(
            status_code=400,
            detail="File encoding not supported. Please use UTF-8."
        )
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"Document analysis failed: {str(e)}"
        )
```

### Batch Processing Endpoint
```python
from typing import List

@router.post("/analyze-batch")
async def analyze_documents_batch(
    documents: List[UploadFile] = File(...),
    nlp_service: HybridLegalNLP = Depends(get_hybrid_nlp)
) -> Dict[str, Any]:
    """
    Analyze multiple legal documents in parallel.
    
    Args:
        documents: List of uploaded legal documents
        
    Returns:
        Batch analysis results with individual document analyses
    """
    if len(documents) > 10:
        raise HTTPException(
            status_code=400,
            detail="Maximum 10 documents per batch"
        )
    
    try:
        # Extract text from all documents
        texts = []
        filenames = []
        
        for doc in documents:
            content = await doc.read()
            text = content.decode('utf-8')
            texts.append(text)
            filenames.append(doc.filename)
        
        # Process in parallel
        async def analyze_document(text: str) -> Dict[str, Any]:
            loop = asyncio.get_event_loop()
            return await loop.run_in_executor(
                None, nlp_service.comprehensive_analysis, text
            )
        
        analyses = await asyncio.gather(*[
            analyze_document(text) for text in texts
        ])
        
        # Format results
        results = []
        for filename, analysis in zip(filenames, analyses):
            results.append({
                "filename": filename,
                "document_type": analysis["document_analysis"]["type"],
                "confidence": analysis["document_analysis"]["confidence"],
                "entity_count": len(analysis["entities"]["combined_unique"]),
                "analysis": analysis
            })
        
        return {
            "status": "success",
            "processed_count": len(results),
            "results": results
        }
        
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"Batch processing failed: {str(e)}"
        )
```

## Dependency Injection

### NLP Service Dependencies
```python
from functools import lru_cache

@lru_cache(maxsize=1)
def get_hybrid_nlp() -> HybridLegalNLP:
    """Get cached hybrid NLP instance."""
    return HybridLegalNLP()

async def get_nlp_service() -> HybridLegalNLP:
    """FastAPI dependency for NLP service."""
    return get_hybrid_nlp()

# Example usage in endpoint
@router.get("/nlp-status")
async def get_nlp_status(
    nlp_service: HybridLegalNLP = Depends(get_nlp_service)
) -> Dict[str, Any]:
    """Get status of NLP services."""
    status = nlp_service.get_service_status()
    return {
        "nlp_status": status,
        "model_info": nlp_service.bert_service.get_model_info()
    }
```

### Database Dependencies
```python
from sqlalchemy.ext.asyncio import AsyncSession

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/medelman17) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
