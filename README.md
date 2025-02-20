# code_base

package com.api.controller;

import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;
import java.util.*;

@RestController
@RequestMapping("/openapi-list")
public class OpenApiController {

    @GetMapping
    public Map<String, List<String>> listOpenApiFiles() throws IOException {
        Map<String, List<String>> openApiFilesByService = new HashMap<>();
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

        // Get all microservice folders under /openapi/
        Resource[] serviceDirs = resolver.getResources("classpath:/openapi/*/");

        for (Resource serviceDir : serviceDirs) {
            String serviceName = serviceDir.getFilename();  // Extract microservice name

            Resource[] resources = resolver.getResources("classpath:/openapi/" + serviceName + "/*.yaml");
            List<String> apiFiles = new ArrayList<>();

            for (Resource resource : resources) {
                apiFiles.add(resource.getFilename());  // Store API YAML file names
            }

            apiFiles.sort(String::compareTo);
            openApiFilesByService.put(serviceName, apiFiles);
        }

        return openApiFilesByService;
    }
}






package com.api.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springdoc.core.models.GroupedOpenApi;

@Configuration
public class SwaggerConfig {

    @Bean
    public GroupedOpenApi publicApi() {
        return GroupedOpenApi.builder()
                .group("public")
                .pathsToMatch("/**")
                .build();
    }
}






<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Microservices API Documentation</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; }
        .grid-container { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; padding: 20px; max-width: 800px; margin: auto; }
        .grid-item { padding: 20px; border: 2px solid #007bff; border-radius: 10px; cursor: pointer; background-color: #f8f9fa; }
        .grid-item:hover { background-color: #e2e6ea; }
        .hidden { display: none; }
        #api-list { margin-top: 20px; }
        #api-list ul { list-style: none; padding: 0; }
        #api-list li { margin: 10px 0; }
    </style>
</head>
<body>

    <h1>Microservices API Documentation</h1>

    <div class="grid-container" id="microservice-grid"></div>

    <div id="api-list" class="hidden">
        <h2 id="service-title"></h2>
        <ul id="openapi-list"></ul>
        <button onclick="goBack()">Back</button>
    </div>

    <script>
        let servicesData = {};

        async function LoadMicroservices() {
            try {
                let response = await fetch('/openapi-list');
                servicesData = await response.json();
                const grid = document.getElementById('microservice-grid');
                grid.innerHTML = ""; 

                Object.keys(servicesData).forEach(service => {
                    const div = document.createElement('div');
                    div.classList.add('grid-item');
                    div.textContent = service;
                    div.onclick = () => showApis(service);
                    grid.appendChild(div);
                });

            } catch (error) {
                console.error('Error loading microservices:', error);
                document.getElementById('microservice-grid').innerHTML = '<p>Error loading data</p>';
            }
        }

        function showApis(service) {
            document.getElementById('microservice-grid').classList.add('hidden');
            document.getElementById('api-list').classList.remove('hidden');
            document.getElementById('service-title').textContent = service + " APIs";

            const apiListElement = document.getElementById('openapi-list');
            apiListElement.innerHTML = "";

            servicesData[service].forEach(api => {
                const listItem = document.createElement('li');
                const link = document.createElement('a');
                link.href = `/swagger-ui/index.html?urls.primaryName=${service}/${api}`;
                link.textContent = api;
                listItem.appendChild(link);
                apiListElement.appendChild(listItem);
            });
        }

        function goBack() {
            document.getElementById('api-list').classList.add('hidden');
            document.getElementById('microservice-grid').classList.remove('hidden');
        }

        window.onload = LoadMicroservices;
    </script>

</body>
</html>
