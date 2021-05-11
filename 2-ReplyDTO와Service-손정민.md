# 05-12 발표자료

## 6.2 ReplyDTO와 ReplyService/ReplyController

---

Reply를 컨트롤러와 서비스 영역에서 처리하기 위한 ReplyDTO 클래스 추가 예제

```java
package org.zerock.board.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Builder
@AllArgsConstructor
@NoArgsConstructor
@Data
public class ReplyDTO {
	private Long rno;
	private String text;
	private String replyer;
	private Long bno;    // 게시글 번호
	private LocalDateTime regDate, modDate;
}
```

ReplyDTO를 Reply엔티티로 처리하거나 or 반대의 경우에 대한 처리는 ReplyService 인터페이스, ReplyServiceImpl 클래스를 작성해서 처리함.

- ReplyService의 기능
    - 댓글을 등록하는 기능(register)
    - 특정 게시물의 댓글 리스트를 가져오는 기능(getList)
    - 댓글을 수정(modify)하고 삭제(remove)하는 기능
    - Reply를 ReplyDTO를 변환하는 entityToDTO()
    - ReplyDTO를 Reply로 변환하는 dtoToEntity()

```java
package org.zerock.board.service;

import org.zerock.board.dto.ReplyDTO;
import org.zerock.board.entity.Board;
import org.zerock.board.entity.Reply;

import java.util.List;

public interface ReplyService {
	Long register(ReplyDTO replyDTO);  // 댓글의 등록
	List<ReplyDTO> getList(Long bno);  // 특정 게시물의 댓글 등록
	void modify(ReplyDTO replyDTO);  // 댓글 수정
	void remove(Long rno);

// ReplyDTO를 Reply객체로 변환 Board객체의 처리가 수반됨
	default Reply dtoToEntity(ReplyDTO replyDTO) {
		Board board = Board.builder().bno(replyDTO.getBno()).build();

		Reply reply = Reply.builder();
						.rno(replyDTO.getRno())
						.text(replyDTO.getText())
						.replyer(replyDTO.getReplyer())
						.board(board)
						.build();
		return reply;
	}

// Reply객체를 ReplyDTO로 변환 Board 객체가 필요하지 않으므로 게시물 번호만
	default ReplyDTO entityToDTO(Reply reply) {
		ReplyDTO dto = ReplyDTO.builder()
						.rno(reply.getRno())
						.text(reply.getText())
						.replyer(reply.getReplyer())
						.regDate(reply.getRegDate())
						.modDate(reply.getModDate())
						.build();
		return dto;
	}
}
```

```java
package org.zerock.board.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import org.zerock.board.dto.ReplyDTO;
import org.zerock.board.entity.Board;
import org.zerock.board.entity.Reply;
import org.zerock.board.reposity.ReplyRepository;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequireArgsConstructor
public class ReplyServiceImpl implements ReplyService {
	private final ReplyRepository replyRepository;

	@Override
	public Long register(ReplyDTO replyDTO) {
		Reply reply = dtoToEntity(replyDTO);
		replyRepository.save(reply);
		return reply.getRno();
	}
	
	@Override
	public List<ReplyDTo> getList(Long bno) {
		List<Reply> result = replyRepository
						.getRepliesByBoardOrderByBno(Board.builder().bno(bno).build());

		return result.stream().map(reply -> entityToDTO(reply)).collect(Collectors.toList());
	}

	@Override
	public void modify(ReplyDTO replyDTO) {
		Reply reply = dtoToEntity(replyDTO);
		replyRepository.save(reply);
	}

	@Override
	public void remove(Long rno) {
		replyRepository.deleteById(rno);
	}
}
```

### 6.2.1 ReplyService 테스트

```java
package org.zerock.board.service;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpinrgBootTest;
import org.zerock.board.dto.ReplyDTO;

import java.util.List;

@SpringBootTest
public class ReplyServiceTests {
	@Autowired
	private ReplyService service;

	@Test
	public void testGetList() {
		Long bno = 100L;  // 데이터베이스에 존재하는 번호
		List<ReplyDTO> replyDTOList = service.getList(bno);
		replyDTOList.forEach(replyDTO -> System.out.println(replyDTO));
	}
}
```

```sql
Hibernate:
	select
		reply0_.rno as rno1_1_,
		reply0_.moddate as moddate2_1_,
		reply0_.regdate as regdate3_1_,
		reply0_.board_bno as board_bn6_1_,
		reply0_.replyer as replyer4_1_,
		reply0_.text as text5_1
	from
		reply reply0_
	where
		reply_.board_bno=?
	order by
		reply0_.rno asc
ReplyDTO(rno=181, text=Reply.......181, replyer=guest, bno=null,
redDate=2021-05-11T09:28:13.868397, modDate=2021-05-11T09:28:13.868397)
ReplyDTO(rno=181, text=Reply.......181, replyer=guest, bno=null,
redDate=2021-05-11T09:28:13.868397, modDate=2021-05-11T09:28:13.868397)
ReplyDTO(rno=181, text=Reply.......181, replyer=guest, bno=null,
redDate=2021-05-11T09:28:13.868397, modDate=2021-05-11T09:28:13.868397)
```

---

### 6.2.2 @RestController

위에서 서비스 계층까지 완료했다면 컨트롤러를 만들어서 조회 화면에서 Ajax로 댓글을 표시 해주어야 함

```java
package org.zerock.board.controller;

import lombok.RequiredArgsConstructor;
import lombok.extern.log4j.Log4j2;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.zerock.board.dto.ReplyDTO;
import org.zerock.board.service.ReplyService;

import java.util.List;

@RestController
@RequestMapping("/replies/")
@Log4j2
@RequireArgsConstructor
	public class ReplyController {
		private final ReplyService replyService;  // 자동주입을 위해 final

		@GetMapping(value = "/board/{bno}", produces = MediaType.APPLICATION_JSON_VALUE)
		public ResponseEntity<List<ReplyDTO>> getListByBoard(@PathVariable("bno") Long bno ) {
			log.info("bno: " + bno);
			return new ResponseEntity<>( replyService.getList(bno), HttpStatus.OK);
		}
}
```

@RestController의 모든 메소드의 리턴 타입은 기본으로 JSON 사용
