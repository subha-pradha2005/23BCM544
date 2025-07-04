package com.example.urlshortener;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;
import org.springframework.stereotype.*;
import org.springframework.http.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.data.jpa.repository.*;
import org.springframework.data.annotation.Id;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import org.springframework.boot.CommandLineRunner;

import javax.persistence.*;
import java.net.URI;
import java.util.Optional;
import java.util.UUID;

@SpringBootApplication
public class UrlShortenerApplication {
    public static void main(String[] args) {
        SpringApplication.run(UrlShortenerApplication.class, args);
    }
}

// 🔗 Entity
@Entity
class ShortUrl {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true)
    private String shortCode;

    @Column(length = 2048)
    private String originalUrl;

    // Getters and setters
    public Long getId() { return id; }
    public String getShortCode() { return shortCode; }
    public void setShortCode(String shortCode) { this.shortCode = shortCode; }
    public String getOriginalUrl() { return originalUrl; }
    public void setOriginalUrl(String originalUrl) { this.originalUrl = originalUrl; }
}

// 📦 Repository
interface ShortUrlRepository extends JpaRepository<ShortUrl, Long> {
    Optional<ShortUrl> findByShortCode(String shortCode);
}

// ⚙️ Service
@Service
class ShortUrlService {
    @Autowired
    private ShortUrlRepository repository;

    public String shortenUrl(String originalUrl) {
        String code = UUID.randomUUID().toString().substring(0, 6);
        ShortUrl shortUrl = new ShortUrl();
        shortUrl.setShortCode(code);
        shortUrl.setOriginalUrl(originalUrl);
        repository.save(shortUrl);
        return code;
    }

    public String getOriginalUrl(String code) {
        return repository.findByShortCode(code)
                         .map(ShortUrl::getOriginalUrl)
                         .orElseThrow(() -> new RuntimeException("Short URL not found"));
    }
}

// 🌐 Controller
@RestController
@RequestMapping("/api")
class ShortUrlController {
    @Autowired
    private ShortUrlService service;

    @PostMapping("/shorten")
    public String shorten(@RequestBody String originalUrl) {
        return service.shortenUrl(originalUrl);
    }

    @GetMapping("/{code}")
    public ResponseEntity<Void> redirect(@PathVariable String code) {
        String originalUrl = service.getOriginalUrl(code);
        return ResponseEntity.status(HttpStatus.FOUND)
                             .location(URI.create(originalUrl))
                             .build();
    }
}
