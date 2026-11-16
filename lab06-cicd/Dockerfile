      - name: Poll until the new commit is live and healthy
        env:
          APP_URL: ${{ vars.PUBLIC_APP_URL }}
          EXPECTED_SHA: ${{ github.sha }}
        run: |
          set -uo pipefail
          APP_URL="${APP_URL%/}"
          if [ -z "${APP_URL}" ]; then
            echo "::error title=PUBLIC_APP_URL not set::Add it as a repository Variable"
            exit 1
          fi
          for attempt in $(seq 1 24); do
            rm -f /tmp/status.json
            CODE=$(curl --silent --max-time 10 --output /tmp/status.json \
              --write-out "%{http_code}" "${APP_URL}/api/status" || echo "000")
            LIVE=$(jq -r '.commit // empty' /tmp/status.json 2>/dev/null || true)
            echo "attempt ${attempt}/24 http=${CODE} live_commit=${LIVE:-none}"
            if [ "${CODE}" != "200" ]; then head -c 200 /tmp/status.json 2>/dev/null; echo; fi
            if [ "${LIVE}" = "${EXPECTED_SHA}" ]; then
              curl --fail --silent --max-time 10 "${APP_URL}/healthz"; echo
              echo "### Deployed ${EXPECTED_SHA}" >> "$GITHUB_STEP_SUMMARY"
              exit 0
            fi
            sleep 15
          done
          echo "::error title=Deployment not healthy::commit ${EXPECTED_SHA} not live after 6 minutes"
          exit 1
