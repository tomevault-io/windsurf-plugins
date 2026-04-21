---
trigger: always_on
description: Weather Flick Batch System Development Rules
---


# Weather Flick Batch System Team Rules

당신은 Weather Flick 배치 시스템 개발의 전문가입니다. Python, APScheduler, 대용량 데이터 처리, 외부 API 연동, 그리고 배치 시스템 최적화에 능숙합니다.

## 🎯 프로젝트 컨텍스트

### 역할 및 책임
- **데이터 수집 시스템**: KTO (한국관광공사), KMA (기상청) API 연동
- **배치 스케줄링**: APScheduler 기반 정기 작업 실행
- **데이터 처리 파이프라인**: 수집 → 변환 → 검증 → 저장
- **추천 엔진**: 날씨 기반 여행지 추천 점수 계산
- **시스템 유지보수**: 백업, 로그 관리, 데이터 품질 검사

### 기술 스택
- **Framework**: APScheduler + Python 3.11+
- **API 클라이언트**: 다중 API 키 관리 시스템
- **Data Processing**: Pandas, NumPy 기반 대용량 처리
- **Database**: PostgreSQL + SQLAlchemy
- **Caching**: Redis 기반 캐싱 시스템
- **Code Quality**: Ruff + Black + MyPy + pytest

## 📁 필수 프로젝트 구조

```
app/
├── core/                    # 핵심 시스템 모듈
│   ├── base_job.py         # 배치 작업 기본 클래스
│   ├── multi_api_key_manager.py # API 키 관리 시스템
│   ├── smart_scheduler.py  # 스마트 스케줄러
│   ├── error_handling.py   # 통합 에러 처리
│   └── logger.py          # 로깅 시스템
├── collectors/             # 데이터 수집 모듈
│   ├── kto_collector.py    # KTO API 수집기
│   ├── weather_collector.py # 기상청 API 수집기
│   └── unified_kto_client.py # 통합 KTO 클라이언트
├── processors/            # 데이터 처리 모듈
│   ├── data_transformation_pipeline.py
│   └── tourism_data_processor.py
├── jobs/                 # 배치 작업 정의
│   ├── weather/          # 날씨 데이터 작업
│   ├── tourism/          # 관광 데이터 작업
│   ├── recommendation/   # 추천 시스템 작업
│   ├── quality/         # 데이터 품질 검사
│   └── system_maintenance/ # 시스템 유지보수
└── schedulers/          # 스케줄러 관리
    └── advanced_scheduler.py
```

## 🛠️ 핵심 개발 원칙

### 1. 배치 작업 기본 패턴 (필수 준수)

```python
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional
from datetime import datetime
import logging

from app.core.error_handling import handle_batch_error, BatchRetryStrategy
from app.core.logger import get_batch_logger

class BaseJob(ABC):
    """배치 작업 기본 클래스"""
    
    def __init__(self, job_name: str):
        self.job_name = job_name
        self.logger = get_batch_logger(job_name)
        self.start_time: Optional[datetime] = None
        self.retry_strategy = BatchRetryStrategy()
    
    async def execute(self, **kwargs) -> Dict[str, Any]:
        """
        배치 작업 실행 메인 메서드
        
        Returns:
            Dict[str, Any]: 작업 실행 결과
        
        Raises:
            BatchJobError: 배치 작업 실행 실패
        """
        self.start_time = datetime.now()
        job_context = {
            "job_name": self.job_name,
            "start_time": self.start_time,
            "parameters": kwargs
        }
        
        try:
            self.logger.info(f"배치 작업 시작: {self.job_name}")
            
            # Pre-execution validation
            await self.validate_prerequisites()
            
            # Main job execution
            result = await self.run_job(**kwargs)
            
            # Post-execution cleanup
            await self.cleanup()
            
            execution_time = (datetime.now() - self.start_time).total_seconds()
            success_result = {
                "status": "SUCCESS",
                "execution_time": execution_time,
                "processed_records": result.get("processed_records", 0),
                "result": result
            }
            
            self.logger.info(
                f"배치 작업 완료: {self.job_name}, "
                f"실행시간: {execution_time:.2f}초, "
                f"처리 레코드: {result.get('processed_records', 0)}개"
            )
            
            return success_result
            
        except Exception as e:
            execution_time = (datetime.now() - self.start_time).total_seconds()
            error_result = await handle_batch_error(
                e, job_context, self.retry_strategy
            )
            
            self.logger.error(
                f"배치 작업 실패: {self.job_name}, "
                f"실행시간: {execution_time:.2f}초, "
                f"오류: {str(e)}"
            )
            
            return {
                "status": "FAILED",
                "execution_time": execution_time,
                "error": str(e),
                "error_details": error_result
            }
    
    @abstractmethod
    async def run_job(self, **kwargs) -> Dict[str, Any]:
        """실제 배치 작업 로직 구현"""
        pass
    
    async def validate_prerequisites(self) -> None:
        """작업 실행 전 필수 조건 검증"""
        pass
    
    async def cleanup(self) -> None:
        """작업 완료 후 정리 작업"""
        pass

# 구체적인 배치 작업 구현 예시
class WeatherDataJob(BaseJob):
    """날씨 데이터 수집 배치 작업"""
    
    def __init__(self):
        super().__init__("weather_data_collection")
        self.weather_collector = WeatherCollector()
        self.data_validator = WeatherDataValidator()
    
    async def validate_prerequisites(self) -> None:
        """API 키 및 데이터베이스 연결 확인"""
        if not await self.weather_collector.check_api_availability():
            raise ValueError("기상청 API 연결 불가능")
    
    async def run_job(self, regions: List[str] = None) -> Dict[str, Any]:
        """날씨 데이터 수집 실행"""
        # 입력 검증 (Early Return)
        if not regions:
            regions = await self._get_default_regions()
        
        if not regions:
            raise ValueError("수집할 지역이 없습니다")
        
        total_processed = 0
        failed_regions = []
        
        for region in regions:
            try:
                # 지역별 데이터 수집

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aicc6) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
