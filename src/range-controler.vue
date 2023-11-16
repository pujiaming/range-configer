<template>
  <div class="interval ">
    <section class="form">
      <div class="settings">
        <label for="num">{{numLabel}}：{{ num }}</label>
      </div>
      <div class="settings">
        <button class="action-button" type="button" :disabled="readonly" @click="num++">增加</button>
        <button class="action-button warning" type="button" :disabled="num<=minNum || readonly" @click="num--">减少</button>
      </div>
    </section>
    <div class="line-wrap">
      <div class="max" v-if="slidable && !infinite">{{ max }}</div>
      <div class="min" v-if="slidable && !infinite">{{ min }}</div>
      <div class="toast" v-if="toastVisible">{{ toast }}</div>
      <div class="line" v-if="slidable && !infinite">
        <template v-for="(item,index) in intervalArr">
          <input v-if="index === 0" :key="0" disabled :value="item.rangeValue" :min="min" :max="max" :step="step" type="range" class="range" list="tickmarks">
          <input v-else-if="index === (intervalArr.length -1)" :key="intervalArr.length -1" disabled :value="item.rangeValue" :min="min" :max="max" :step="step" type="range" class="track-hidden">
          <input
            v-else
            :key="index"
            :min="min"
            :max="max"
            :value="item.rangeValue"
            :step="step"
            type="range"
            :disabled="readonly"
            class="track-hidden"
            @input="handleInput(index, $event)"
            @mouseup="handleUp"
            @change="mutationRange(item,index)"
          >
        </template>
      </div>
      <section class="range-input-wrapper" :style="slidable && !infinite?'margin-top:40px;':''">
        <div v-for="(item,index) in interval" :key="index" class="range-input-block">
          <section class="flex">
            <label for="range" style="white-space: nowrap;">{{ label }}：</label>
            <div class="range-outer">
              <input class="range-input no-arrow" name="range" type="number" :readonly="readonly" :value="item[from]" placeholder="区间起点" @blur="handleRangeInputFrom(index, $event, interval)">
              <div class="sym">至</div>
              <input class="range-input no-arrow" type="number" :readonly="readonly" :value="item[to]" placeholder="区间终点" @blur="handleRangeInputTo(index, $event, interval)">
              <img v-if="num>minNum && !readonly" class="delete-action" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAAAAXNSR0IArs4c6QAAIABJREFUeF7tfQuYZUV17lq7W2hnBmeE3AS9NxcxPIZoiA/kkQsoAoNXUESMgygaQnRQdAJM71XdY4yHBKa7avfMwHCJDImAgkY0CQJyBcHgg5cE+cAkAgPewWgUuQIzMtM2zum97rfGPd5mZr937XP2Oafq+87X3V9XrVr1V/2n9q5aDwRXHAIOgUQE0GHjEHAIJCPgCOJWh0MgBQFHELc8HAKOIG4NOATKIeB2kHK4uVYDgoAjyIBMtBtmOQQcQcrh5loNCAKOIAMy0W6Y5RBwBCmHm2s1IAg4ggzIRLthlkPAEaQcbq7VgCDgCDIgE+2GWQ4BR5ByuBVuNTExsZ/nefsAwEL5IOJLot+3/5zzt8jezMy/kJ8AsP3nnL83h2H4w/Hx8ccLK+EaFEbAEaQwZMkNtNb/zfO8AwBg/zAMD0DE/QFg+98A4FnsSkSFAPAYAGxg5sc8z9sgf4dhuEEp9WPLfQ2sOEeQClOvtT4UAP4IEd8IAG8CgEUVxNlsugkAvsHM3wSAu5VS99kUPkiyHEFyzrbWeg8AEEIciojys0mEyBrFDsIIUbZ/lFLPZTVy/wdwBElZBevXr3/Rpk2b3oaIbwMA+ezVJ4vmaQC4iZlvWrRo0U3Lli3b1ifjsj4MR5AYSIMgOC4Mwx3E2Nc66s0SuFGI4nneTb7v394s1bqvjSNINAda60MQ8WRmFmL8YfenpvMaMPNDiCg7yw1Kqfs7r0Hzehx4gmitj/E870xmPsP29CDiFmZ+BADk86j8zsxPIuKM53kz7XZ7+8+hoaGZLVu2zACAfKSMLFiwYGR2dnYkDMOR4eHh7T+ZeQQR90bExQBwIAAslt+ZeUENul8ThuFVSqk7bMvuJXkDS5AgCGS3OBMATrY0YQ8j4t1hGD7oed4jYRg+0qnj1uh4eXEYhos9z3sNM/8RABxkaVw3IOJVvu/fYEleT4kZOIJorc9ARCHGMRVmSr7p7waAe5n5O4h4HxE9WUGe9abGmL2ZWU7cDgOAw+U4WnamCh3dwcyyo1xTQUbPNR0YgkxOTr7b8zwfAA4pOUvPAMA/AcD109PTd7daLTk67ZnSarUWzZs3T0hyCgC8EwD2LKn8/WEYBmNjY18s2b6nmvU9QbTWByLiGAD8ScmZ+RoibieG7/tPlZTRqGZBEPy2EIWZhShLSip3NTNPKqUeLdm+J5r1NUGCIDiXmccBQBZEkfKwEIKZr+/305zo9E52FfkUfW95ChEnfN+/uAi4vVS3LwkiJ1OIKMQ4vuBkiGnG1UR0dcF2fVE9CIKzmPksADii4IBuY+aJfjzx6iuCXHbZZQu2bt36SQAYLTjBNyLi1b7vX1+wXV9W11q/FxGFKEUPMqbmz59/wTnnnLOlX4DpG4JMTU0dGYahKfjtdy0zX62U+nq/TKjNcWit3xUR5S0F5N7jeR6Njo7eWaBNY6v2BUGMMR8DAA0AL86DNDPLCUzQ7+8XebDIU0dr/XYA8BHxyDz1AeCXAKCI6NKc9RtbracJsmrVqr2Gh4dl1/jTnAjLy/ckEX02Z31XbQ4CWusViCiPr3vnBObKdrtNK1euFOPIniw9SxCt9bGIKOR4XU7kp9rt9mQvT1bOcdZazRjze9E73tk5O3qAmalXH2N7kiDRN5mQI9NLj5lvDcNwcnx8/Bs5J9RVy4HA5OTkCdHF67E5qocRSVbnqNuoKj1HEK31WkQ8NweKbWYeU0r13KTkGFtjqhhj5JEryKMQM1+slDovT92m1OkpghhjxA7ofTnAuy8ix0BboubAyUoVY4ycck0BwKtyCLyWiKxbTufot1SVniGI1voWRDwhxygvHxkZUcuXL5doIK50CAGxKEZEIcnSrC7lsVcpVeToOEtkbf/vCYIYY8R55/UZKDwThuHY2NjY39aGlhOciYAx5i8A4K8zKwJ8l4jKGo7mEG+nSuMJYox5AgAknlRaEVdROXd/wA4sTkoVBCJfG3n3kxOvtPJDInpFlb7qbttogmittyLivAwQPk9E760bKCe/GAJa64MQ8fMA8Jq0lsw8rZSaX0x652o3liBBEDzLzKlxpiJPt7yXhJ1D1fW0HYG1a9e+bNu2beIqIA5biQURN/m+/9ImwtZIggRB8CAzZwVOuJyIPtxEUJ1OL0TAGCN3UBJcL40kD/m+n7rbdAPXxhHEGCO+z2L7kwam9n1fnKBc6REEjDG35nDOupGIbMUIsIJMowhijFkHAGJ4mFaWEdEVVkbvhHQUAWPMZwDg/RmdXkpEyzuqWNqXcVMUMcbIbWyqHwciHu+CmzVlxsrpobVuIaL47KSVKSKS+AFdL43YQYIguJCZP56BxruI6B+7jphToDICQRAYZk4lACJe5Pu+3Kl0tXSdIFpr8TMQw8O08hEi+lRXkXKdW0UgCILLmPkjaUKZebTbtnRdJYgx5iQJopwB0gVKqZbV2XHCGoFAEAQSGeUDKcqIFfCSbprKd40gknFpeHj4q8y8XwpAVxKR+Ea70qcIGGP+AQBOTRneA+12e0m3/Hi6RpAgCG6UQNEpwNw+MjJy0vLly5/v07XhhgUAExMTLx0aGpKj/aOa+EXZFYLkeEn7AQD8TyKSFGOu9DkCUXC/mzNst5Z3w8e94wTRWp+JiFemzTkivmNQgyX3ORcShxcZOH45Zfy/9DxvSaejpXSUIFrrwxHxlii7axIWnyCiCwd1oQzyuHOYyt8zf/78JZ2Mu9VRguQwN7iOiE4b5EUy6GM3xnwhw+mqo5eIHSOI1vpsREy7y/h3Zn5Lp3JqDPpCbOr4I89EecpIdN9l5jd3KsxpRwgSRRO/K+NIV17KBRhXBhyByMf9qykw3EZEZaPSF0K3UwTJMi3wiUj8mV1xCGxHICtaCiKe14mo8rUTJHoxvyslhtXXieg4ty4cAjsjYIwRV+qkuFtPMfPRdecnqZ0gWTelYRi+ZWxsTHwFXHEIvACBKDhd2mO3pKqQdHq1lVoJEgTB6cz8uRTtnVdgbVPbH4KNMXKwkxjmNAzDpXWmg6uVIMaYf0nJCShJL48kIrk1d8UhEItAFAtYUikkBcy+n4jeUBd8tREkyiabGEW9CabMQRC8mpkld6HE3Ho9M/8UACR00KODaEEcBMHBYRhKLABJWfe94eHhR1asWPGjuhZfXrlRLObEQxxmfn9d2XdrI4gx5p+TMhQx851KqTTjtLzYla5njLkIAP4cAJJCzki2KTld6/sdTojBzOLRGXd0+jkiyhPutfRc5Gmotf52Sn6SO4jozXnkFK1TC0Gy7GqY+WSl1I1FlbVVPyLHyhzyvo+IS33f/7ccdXuyiqTHRsR1iPg7KQN4hoj26uYAJYkPIorVb2ypy36vFoIYY8ToLCk6xfVEJOmHu1KivCJyfJi3PBy9CP5r3ga9Us8Y8x4AkOBueUrXgykYY+TyMCmm7w1E9I48AylSxzpBogyz8ngVWyKLzNuKKGmzbhAElzLzRwvKfDQiyUMF2zW2ehAE72NmiZafuyDiGb7vX5u7geWKUc7ELyWJrcMExTpBgiD4LDPHhrdn5i8opeRbq2vFGCMpEd5UVAFE3MDMS4nowaJtm1Zfa/0ByepbQq+/J6LTS7Sz1iTt3RYRr/F9PyusUCFdrBIkSkovR7uxJQzDN46NjX2rkIaWKxtjJC3CHmXEMvPj8k7Sy0GyjTESqvXTJce/QSl1YJm2ttpEKaoTdzFmfoPN5KxWCWKMkbD3SaFaar/1zDMJZXeQObJ/4Hne0tHR0e/m6a9JdSYnJz/oeV6VoHvPEdFLuj0mY8zdKem+LySiT9jS0SpBtNYPImJsTF1mPkwpdZ8txcvKyRlmKEv8xtnZ2aXj4+OJu2WWgE7/P4e7QR6VvkFEx+SpWGedIAjOYua/i+uDmR9SSlmL8WuNIEEQHMfMSS/fjUm7ZYyRrLj3AMBuFSfxCXknaQLps8ZhjDkHAP5XVr0c/19NRKnRL3PIsFLFGPN9ADgoTpjNCJzWCKK1vgQRY2OqIuJ7fN8XT7FGlDxhTvMoysz/Id5vSql789TvRh2t9XJEvMRC30/Mzs4eMz4+LgmNul7S7rKYeZ1SSi6BKxcrBFm/fv2LNm/e/CgA7Buj0ZPMfIBS6rnK2loSkMdrrUBXP4ouE2VXalQxxkhG2TWWlDqTiMqcfFnq/oViMg6ENi5cuPDAZcuWbavauRWCaK3fiYixcXOZ+dNKqT+rqqjt9lEGpC8CwKstyP4xAJxGROL30oiS5XBURElEbPm+f0GRNp2omxbjgJlPVUpJ8p5KxQpBjDFXAYAY/e1SEPGdvu+LXVPjytTU1OIwDK8DgIMtKPeT6DJRLE+7WrTWChEnbSiBiH/p+36epJw2uiskIwiCZcx8eUIjK6emlQmybt26l8zMzPwfAIiz1Xlienr6wFar9atCI+9g5dWrVx/Qbre/mHT6VkQVRPzp7Ozsad286zHGiI2ZGGLaKI0OwSSxDpj5YQDYM2awT4+MjLyyajrwygTJ8Pq6jIiKmnXYmNhCMowx+yPidcz82kINYyoz88+iy8RvVpVVtH0QBJ9g5r8q2i6h/seJaJUlWbWJMcZI2u/YR3gb3qqVCZKWEMXzvLeOjo6mRaeoDbiigiPHHHknkWPgquWp6MVdcvN1pORMTJNLF2ZeqZSayFW5y5WMMW8FAAlbukth5sqZASoTJAiCW5j5hBj9nh8eHn75+eef/0yXMczd/cTExCuHhobkncRGgvufRyRJNNzMrVhGxQwLhkLdMPOYUkoXatTFymvWrNmz3W7/BAB231kNRLzV9/0k699cWlcmiDFmMwDEmR/cRURH5tKiQZXWrl37im3btslOYsON8+mhoaGlK1as+HpdQ9Rar0LEcUvyFRFlJTOy1JU9McYYORj5HzESf0FEC6v0VIkgk5OTR3qe9+0EBS4honOrKNettlNTU/tEp1uHWdBB8r3LZaJ1E39jjHzTkwUdRUTPxiYzxlwceYfuAkUYhkeNjY2VPlmsRBBjjKRijn1WbdrtedFFtHr16t+dnZ2VneTwom1j6m+SG3ci+poFWdtFGGPER3uFDXlNiA9QZRxBEJzGzH+fIGOciEofeVciiNb6K4h4YpxizPxflVLybNizJbpxF5IcYWEQYmYvJKkcXlVrvRYRbe3O5xPRWgvj65oIrfXLEfE/E9bhzUopSfVXqlQiiDFGvhnjnvEeI6IDSmnUsEYR+EKSuGfcoto+F5nKlz7Zy5lLPq9e5xKRDTutvP3VVs8YswEA9o/pYDMRLSrbcWmCRN+usSFhEPEq3/fFMacvykUXXfSy4eFhuUy0ceiwNTrdij2aTANMa30ZIqZmhs0LODP/uVJqXd76Ta8XBMGVzBwbZXFoaOi/lw1fVJogQRC8mZljT2eY+WNKKRvm1Y2Zl4svvvh3nn/+eSHJ0RaUmo4et76SV5YxRkwqluWtn1YPET/m+35fzY/W+qOIeGncuKuYv1chSKIdTBWFbCyAumREpg3yuPXGqn0w80xkKp8Z/igIgiuY+YNV+4zaf5SILrMkqzFi0vyRmPkcpdTflFG2NEG01qsR8fy4Tpn5d/s1Ec7U1NRvhWEoJLHhWfd89LiVGO/JGCP+47YeVz9CRGlJjMqsoUa0SXvkr+IfUpogxhj55tsljTMibvF9v1RQhEYgnUOJVatW7SXvJABgI5rfrxDxtDiL5yAIrmbmD+RQKbMKM39YKZVk+ZrZvhcqBEHwHDMv2FlXZr5VKVXqRr0KQR4BgLgIF7UGE27KREX5vYUkNnKbtKPTrd/4LxhjJGaVrZCfy4ioSrCGpsCeqkdKsPSNRPTKMoOoQpDZhKQ4jfE/LwNIkTZr165dFJmlHF+kXcKL5GwYhqcppf7BGCPRDq3ED0PED/m+LxavfV/SvlSmp6d3L+N2UYogExMT+w0NDT2WgHijfQhsrxLxh5HTrQSDzaLdMSJ+iZnfXbRhAun+zPf9UjGwbPTfaRlpaaQR8Q/KxFguRZC0+LbM/MfyLdhpcLrZn9Z6D0SUx61Sz7k16X4WEV1Zk+xGik0LTVrWN6QsQRJ90KsahzUS+RxKBUEwn5mFJOKf0O3SqAALnQIjzXi2bCaqsgQ5ExFjv51sh37sFLg2+mm1WvNe/OIXy2VirH2ajT6yZEhCIKXUZ7Lq9eP/MyKdfJCIYoPNpWFRiiDGGIk5JCbGu5Syz3r9MmGtVmtk3rx5spPscgRe9xjrzLRUt+425EcZw2LTVJS1WC5LkL8EgNgwMLOzs/uPj48/bmPAvSpj3bp1u8/MzAhJ3t6pMXQ7NUGnxpnWT8bh0V8R0SeL6lmWIIm+CP18i14E3CiYnpDEelKXnfVAxPf6vp83EU6RYfRU3bTbdAAo5cBXliCJkSQ8z/svo6OjP+8pZGtSttVqDUePW6fU1IWIPZ2IkpyFauy2eaIjM6D/m/DoX8rCvBRBgiCQEDmxZ/XT09N7tFqtLc2DrzsatVotLyLJqTVoINEcJciEKwDQarUWzJs3LzbELSL+k+/7heegLEGSIpnA9PT0i1qtVtvN2AsQQK21nG69yxYuZY8tbfXfRDnRjp0Uj/frRFTYLMgRpDMzbZ0g7r1j14nLIMj/JqLCx+9lCeIesXISq+ZHrFJn+zlV77lqaY9YAFAqv2IpgqSFe3Qv6f9/XXXiJb3Xo8fYZGHaSzoArCeis4v2V5Yg7pg3A+lOHvMCwAk2QwoVXURNqZ8RJyHwfb9wDLGyBHEXhSmrohsXhZ7nHTU6Olo6QFpTFnkVPeqwMi9LEGdqkjCT3TQ1QcTDfd//TpVF1sttM0xNSkVxKUUQrbUzVoxZSU0wVgSA1/dyHvcqBM0wVixl4VyWIM7cfaeZbJK5+6AajKaZu5fNdFaWIMci4u1xbHcOU1W+A+219TzvVaOjo5IqeWBKmsMUMx+nlCocZb8UQep4GerVWbTtcgsA4o35xzbwGDSSpLnclrUyL0UQmTxjjAvaUF/QBrECdiQp+C2RErQhJKKhguK2V69CEBf2Z2iozrA/jiQFV3RK2J9HiWhxQXGVCeICx9UcOM4Y40hSYFUnBY4DgJuIqJTzWukdxIUe7VjoUUeSHCTJCD26RilVKtlQaYKkJXF3wauzZ7RI8Gq3k2TjmRa8GhHP9n1/fbaUXWtUIYhLf1AG8V+3KZP+wO0kKXhnpD841vf9UtmGSxMkwzDs077vxyZ3L7+muteyKQl0bO0kiCgRHF/dT/ckQRD8HTOflXA3VzrbQGmCREe9zwJAXHqrh4no97u3pO313MAUbLZ2kl95nvfafiGJMUYuRQ+KmflNRPTSsiuiKkGuT4na8TIierKsYk1o19QknrZ2EgDY6nneob1OEmPM3gDw04Q182UiKh00oxJBtNbnImJshlRmPlkplZk9qQlEiNOhB9JA29pJnvU878heJonW+u2IGJuEiJnPU0rFBjnMs/aqEuRQREwyr15FRB/Po0TT6kxNTe0ThqFECznMgm7PMvNSpdRtFmS9QITFneQpz/OO6VWSGGMuAoCVCe8fhyml7iuLfSWCZLyH/DMRHVtWsW61W7t27SuinB9vsKDD00NDQ0tXrFhR2Egub98WSfKfnuct6UWSGGME37hsX5XeP2QObBAk6T1kZnp6+mWtVktyqfdEmZiYeOXQ0JDsHIdYUPjnUf7BUseLRfq3SJKNnued1EskabVai+bNmyfvHyMxmFV6/7BCEK31OCKuiptQz/PeOjo6+tUik92tusaY3wMAea5/nQUdnorI8Q0LsnKJsEUSRNyAiKf0CkmMMZJuIjbnPDOvVEpN5AIwoVLlHSQtmQ4AXEZEH62iYCfaGmP2R0QJZfTaqv0x88+EHET0zaqyira3RRIA+HfP897dCyRJi7BT1gdkLu42CCLZlTYCwF4xE/rE9PT0gWVywxVdHGXrr169+oB2uy1RD/+wrIwd7RDxp7Ozs6eNjY19q6qssu0HiSRR3vqHAWDPGLyeZuZ9lVKxoUjz4luZINKRMeYqAPiTuE7LujrmHUCVelNTU4uj06qDq8iJ2v4kCgfa9cgig0KSNHtAALiaiM6sOq9WCKK1TvRRZ+ZPK6UaZ3aitT4oyiv46qogAsCPAUACSd9lQZYVEYNAEmPMrQCwJA4wZj5VKfWbtNplQbVCkChI2qMAsG+MIk8y8wFVt7qyA4xrF92Q3wIAr7Ig90fRC/k9FmRZFWGRJN/bbbfdlpx77rk/s6pgBWEZEUw2Lly48MBly5YlBbLO3bMVgkhvWutLEHF5wmPWe3zf/0JurWquaIxZAwDnVe2Gmf8DAOQS8N6qsupqb4skiHiN7/vvr0vPonIzLgfXKaUkdlvlYo0gafb4AHAtEZ1RWVsLAqL3Dnmxq1qeiG7IS9/SVlUgb3uLJHmn7/ty79X1kmKcCDb9kawRJNpFHkw6DfI875DR0dHvdhtZrXULEQvnqttJ742zs7NLx8fH/6Xb48nbvw2SMPMFSqlW3j7rqmeMkQMhORjapTDzQ0qp19jq2ypBjDF/DQB/kaDcFUS0zJbiZeUYY+4AgDeVbQ8AP/A8b2kTyF50DBZI8iUiis0sVlSXKvWNMXIB+8YEGRcS0SeqyJ/b1ipBMl6coAm7SBWCMPPj0SXgA7YmoNNyKpLkOiI6rdM6z+0vCIJTmDnxdIqZ36CUut+WjlYJIkoFQfBZZk563+j6LpJ285oGqphgyDsHET1oC/xuySlLEkRc6ft+JdONqmM2xohZe2yEkjoOEqwTRGt9DCImGuh1exdJe35NmbxHo0vAh6pOcFPalyRJV/OQZJg1ATO/WSklj9DWinWCiGbGmC8DwMlNfBdZ++toiAJi3he5hyNy/Ks11BsiqCBJDBGpbqqeEjlR1LqBiKznpK+FIEEQnMzMQpLY0u1dRGv9DkTMc1z5/egS8N+6uTDq7DvNTGhOvw8sXLjwcBsXb2XHkvV+i4jv8H0/1quwbJ/SrhaCRLuIPGYdE6ccM39RKbW0iuJV2xpjJPatmLcnlatnZ2cvGB8ff6JqX01vHz12ytH3K3bSVUxoLpmenl7XbYNTrfV1iJh0gnYHEcU5TFWGvjaCaK3PQMTPpmj4ASJK+3/lwWUJmJyc/APP805n5hMQ8UAAEKtkudv4JhFdndW+n/4vGWJHRkYORsSDPc/bhojfY+bv+76/tdvjNMbIDf5nkvRg5vcrpa6pQ8/aCBLtIrLYkrzzHm6320etXLny6ToG5mT2BwKrVq3aa3h4+NsJIX1kkPcTkQ336FjAaiVIEATy7fy5lKmaIiK/P6bSjaIOBIwxAQCMJsmODlDSHpUrqVUrQaJdRBLCnJqk5ezs7DHj4+Mdc02thJZr3FEEJiYm3jQ0NJR2bGvF5yNtULUTRGt9OCKKn4QXpwgz36qUektHkXed9QQCWutbEPGEBGWfYuajlVLiZlFbqZ0gonkQBIaZEx+lmHlUKbW6tlE6wT2HgNZ6BSJOJSmOiOf5vl86IFxeQDpFkN8GgLuYeb8ExdrMvMT2LWheEFy9ZiEQWWN8DQCGEzS7jYhiPQltj6QjBBGltdZnI+KnUgZw38jIyPHLly//he1BOnm9g4AkRZ2ZmZEolIcmaV2HSUniTtVJ6NJ8iCM9LieiD3dSJ9dXsxAwxsiX6NkpWnX05LNjO0i0i8gLu/iCL0wCIAzDD42Njf1ts6bNadMJBCYnJz/oed4VKX3dM3/+/CXnnHPOlk7oI310lCARSc5ExCtTBvgMABxPRD3rc9GpyeunfowxEtFSHq3iYlzJUH8ZxQ7uaFiljhNERpp1qgUAtxPR8f20ANxY0hEwxgg5jkuptZyILu00jl0hSESSG5n5bYkvR4if8X0/Nhhdp0Fy/dWLgDFGrC1OT+nlSiKKTa9Wr2ZdeMTaMaCJiYn9hoeHv5py9CtVLyWi2FBCdQPj5HcGgSAIrmTmtAiID7Tb7SXdstnr2g4i8BtjTpIk7xlTYdUJvzPT7nrJg0COE6swuh+rLb9Klp5dJUj00u4joklTtE5z5iyA3P/rQSAIgklmTvVQbIKFRdcJEpFkLSKemzYVEm/L9/3v1TNdTmonETDGfAgA1md8KV6slKoc/bLquBpBkOhxSxxe3pc2ICJqjL5VgR/U9hkROHfA0phInI1acBnWm9vBC8Nwn7GxMYmJ60qPIWCMEbcHcX9ILE2z7m4UQaKdRIJ+vT4NxDAMjxobG+vohVGPrcXGqWuMEROiv8lQ7LtEZCM/pLXxN44gEUkkUMI+Ge8k7/V9//PWkHCCakMgZzzkHxLRzkEjatMpr+BGEkSU11pvRcR5GQMZJ6LJvIN19TqLwLp163afmZmRXeNPMx6rppVS8zurXb7eGksQUT8IgmeZeVHGUD5FRB/JN1xXq1MISGLU6JEqzXxEUhVs8n3/pZ3Sq2g/jSZIRJIHmTkrwebtAKCcgWPR6a+nfhQ4UDxEJbV2YkHEh3zfzxvhsh5lM6Q2niDRO0liwOI543smDMMxZyrflXX0m06NMZL+QtJgZJUbiSgpPG1W2479vycIEpFkHQB8LAcyl4+MjCjnmZgDKYtVoryP4kOeJ2Jmz9jY9QxBIpKkxkiaM9/3MfOY83G3yIAUUcYYiUoj5MiTFLWjHoFVEegpgkTvJBcy88dzDFwCQQhJXLSUHGCVrWKMkaBu8sWVWRDxIt/3kzKQZbbvRoWeI4iApLXONHDcAabczIZhOOmC09ldXpOTkyd4niehnI7NI5mZSSmVi0h55HWqTk8SJHrcOgkR12b4k8zFcardbk92y6+gUxNadz/GGDmZkl0jLbDCb9RAxMeZ+Twi+krdutUhv2cJImBETldr0jwTdwJN0j9PdjuqfB0T2QmZUTA3IcfeefpDxJva7fb54+Pjj+ep38Q6PU2QHYDm8HF/AfaSn2RoaMj0YqbabiwirbXkBJTH2iPz9o+6BSvWAAAD6ElEQVSIge/7lLd+U+v1BUGi9xKJlrI2LaRQzCRcIWFmHFHil+fU1NTxYRiKkeEpBRbwZnmkUkrF5jEvIKcRVfuGIBFJJO7WBQBQNCylI8qc5Tg5OXk0In4YEYumfP4aM39SKXVvI1a3BSX6iiA78JAwp57nrSjwAr+j6UATRWt9qBADAApFk5EX8TAMVyulLrewJhsloi8JIggHQSABs0eZeUVS6oWUmbgWEW8Ow/BmpdRzjZoxy8porffwPO9EZj4xy6MzpusQEeWeacr3/acsq9YIcX1LkDm7iTx2yclLYhKflJl4kplv9jzv5q1bt97c7USWtlZMq9Xabf78+SeGYXgiIgoxcp1K7dT/PzLzVD89TsXh2/cEmXPSJengJAhAWY81ceLaThbP875z/vnnS4jUnilr1qzZMwzDw4QUACCfss5J98thyKA4qw0MQebsKJJ9VwKVxaaozrnin5fkkfJBxHvDMPyWUuonOdt2pJrW+uWe5x3NzIdHXwryxbB7hc7vYOar6somW0GvWpsOHEHm7CgnRxH9bJlcP4aId4Zh+IDneY+EYfiIUkryjNdexJLW87zFYRgu9jzvdcws9xXisGSj3ICIV/m+Ly4HA1cGliBzdpRjPM87k5nPsD37iLiFmR8BAPk8Kr8z85OIOON53ky73d7+c2hoaGbLli0zACAfKSMLFiwYmZ2dHQnDcGR4eHj7T2YeQcS9EXExAEhe98XyOzMvqEH3a8IwlB0jLYmm7W4bJ2/gCTKHKIcgouwqb5MgdY2bqQ4oxMwPiXkIM9+glJJHyIEvjiAxS0CCm4VhKESR6PP79vkq2cjMN3med5Pv++K67MocBBxBUpbD+vXrX7Rp06YdRBGy7NUnq+dpCRouxFi0aNFNy5Yt29Yn47I+DEeQnJBKcsnp6ekjEPEIz/MOZ+YjAOAlOZt3u9ovEPGeMAzvZeZ75s2bd49zSc43JY4g+XCKrTU5OXmk53lHyqlRZOmamHuxQjdlmorB4J3RqdqdLgplGQh/3cYRpDx2u7SMjlsPkCPWMAwPQEQ5at3+dwlzlyzNQgB4DAA2MPNjnudtkL/DMNzQqePlLAX74f+OIB2aRXHu8jxPwqnKLrMQEeXxTH7f/nPO36KR7ACSL34zAGz/OefvzWEY/rCXnZA6BLmVbhxBrMDohPQrAo4g/TqzblxWEHAEsQKjE9KvCDiC9OvMunFZQcARxAqMTki/IuAI0q8z68ZlBQFHECswOiH9ioAjSL/OrBuXFQQcQazA6IT0KwKOIP06s25cVhBwBLECoxPSrwg4gvTrzLpxWUHAEcQKjE5IvyLw/wBH0qKb+9PNIQAAAABJRU5ErkJggg==" @click.stop="deleteItem(item, index)">
            </div>
          </section>
          <section v-if="purpose" style="margin-top: 10px;">
            <slot name="purpose" :row="{item,index}" />
          </section>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
export default {
  name: 'InputRange',
  model: ['update:configArr'],
  props: {
    configArr: {
      type: Array,
      default: function() {
        return []
      }
    },
    step: {
      type: Number,
      default: 1
    }, // 拦截阶梯值等于最大值
    label: {
      type: String,
      default: 'Range'
    },
    purpose: {
      type: Boolean,
      default: false// 是否使用插槽来表示满足各个区间条件后所能达到的目的
    },
    aimAttri: {
      type: Object,
      default: function(){
        return {
          aimrow: 0
        }
      }// 目的字段，默认为aimrow，用于插槽目的字段的绑定
    },
    readonly: {
      type: Boolean,
      default: false
    },
    minNum: {
      type: Number,
      default: 1
    },
    addStep: {
      type: Number,
      default: 1000
    },
    infinite: {
      type: Boolean, // 是否默认以0结尾
      default: true
    },
    from:{
      type: String,
      default: 'from'
    },
    to:{
      type: String,
      default: 'to'
    },
    numLabel:{
      type:String,
      default: 'range amount'
    },
    slidable:{
      type: Boolean, 
      default: false
    }
  },
  data() {
    return {
      toast: '',
      toastVisible: false
      // default: [
      //   {
      //     from: 0,
      //     to: this.addStep
      //   }
      // ]
    }
  },
  computed: {
    defaultArr:function(){
      return [
        {
          [this.from]: 0,
          [this.to]: this.addStep
        }
      ]
    },
    rangeMount: function({ interval }) {
      return interval.length
    },
    interval: function({ purpose, aimAttri }) {
      let arr
      if (this.configArr.length < this.minNum) {
        arr = JSON.parse(JSON.stringify(this.defaultArr))
        if (purpose) {
          arr = arr.map(item => {
            return {
              ...item,
              ...aimAttri
            }
          })
        }
        this.$emit('update:configArr', arr)
        return arr
      }
      arr = JSON.parse(JSON.stringify(this.configArr))
      if (arr[0][this.from] >= arr[0][this.to] && arr.length > 1) {
        arr[0][this.from] = arr[0][this.to] - this.step
        this.$emit('update:configArr', arr)
        return arr
      }
      if (purpose) {
        arr = arr.map(item => {
          if(Object.keys(aimAttri)[0] in item){
            return item
          }else{
            return {
              ...item,
              ...aimAttri
            }
          }
        })
      }
      return arr
    },
    intervalArr: function({ interval }) {
      let arr = []
      arr = interval.map(item => {
        return {
          rangeValue: item[this.from]
        }
      })
      arr.push({ rangeValue: interval[interval.length - 1][this.to] })
      return arr
    },
    num: {
      get: function({ interval }) {
        return interval.length
      },
      set: function(val) {
        let copy = JSON.parse(JSON.stringify([...this.interval]))
        const oldVal = copy.length
        if (val > oldVal) {
          if (copy[copy.length - 1][this.to] === 0) {
            copy[copy.length - 1][this.to] = copy[copy.length - 1][this.from] + this.step
          }
          let obj = {}
          if (!this.infinite) {
            obj = {
              [this.from]: copy[copy.length - 1][this.to] + 1 * this.step,
              [this.to]: copy[copy.length - 1][this.to] + this.addStep + this.step
            }
          } else {
            obj = {
              [this.from]: copy[copy.length - 1][this.to] + this.addStep,
              [this.to]: 0
            }
          }
          if (this.purpose) {
            // obj[this.aimAttri] = ''
            obj = {...obj, ...this.aimAttri}
          }
          copy.push(obj)
          copy = this.checkQueue(copy, copy.length - 1, this.step)
        } else if (val < oldVal) {
          const keys = Object.keys(this.aimAttri)
          let obj = {
            [this.from]: copy[copy.length - 2][this.from],
            [this.to]: copy[copy.length - 1][this.to]
          }
          keys.forEach(key=>{
            obj[key] = this.interval[copy.length - 2][key]
          })
          copy.splice(copy.length - 2, 2, obj)
        }
        this.$emit('update:configArr', copy)
      }
    },
    min:function(){
      return this.interval[0][this.from]
    },
    max:function(){
      return this.interval[this.interval.length-1][this.to]
    }
  },
  watch: {

  },
  created() {

  },
  mounted() {
  },
  methods: {
    mutationRange() {
      // this.intervalArr.sort((a, b)=>a.rangeValue - b.rangeValue)
      const keys = Object.keys(this.aimAttri)
      this.toastVisible = false
      const copy = this.interval.map((item, index) => {
        let obj
        if (index !== this.interval.length - 1) {
          obj =  {
            [this.from]: this.intervalArr[index].rangeValue,
            [this.to]: this.intervalArr[index + 1].rangeValue - this.step,
          }
        } else {
          obj =  {
            [this.from]: this.intervalArr[index].rangeValue,
            [this.to]: this.intervalArr[index + 1].rangeValue,
          }
        }
        keys.forEach(key =>{
          obj[key] = item[key]
        })
        return obj
      })
      this.$emit('update:configArr', copy)
    },
    handleInput(index, event) {
      const newVal = Number(event.target.value)
      this.toastVisible = true
      this.toast = event.target.value
      const prevVal = this.interval[index][this.from] // 获取变化前的值
      const isHave = this.intervalArr.findIndex((item) => item.rangeValue === newVal) >= 0
      if (isHave) {
        // 如果新的值与rangeValue相等，重置为变化前的值
        this.$set(this.intervalArr[index], 'rangeValue', prevVal)
      } else {
        // 否则更新rangeValue
        this.$set(this.intervalArr[index], 'rangeValue', newVal)
      }
    },
    handleRangeInputTo(index, event, arr) {
      const copy = JSON.parse(JSON.stringify(arr))
      const newVal = Number(event.target.value)
      const prevVal = this.interval[index][this.to] // 获取变化前的值
      if (newVal < 0 || newVal % this.step !== 0 || (newVal === 0 && index !== copy.length - 1)) {
        copy[index][this.to] = prevVal
        this.$emit('update:configArr', copy)
        return
      }
      copy[index][this.to] = newVal
      if (copy[index][this.from] >= copy[index][this.to] && !(index === copy.length - 1 && newVal === 0)) {
        // if (index !== copy.length - 1 && newVal !== 0) {
        //   copy[index].from = copy[index].to - this.step
        // }
        copy[index][this.from] = copy[index][this.to] - this.step
      }
      const res = this.checkQueue(copy, index, this.step)
      this.$emit('update:configArr', res)
    },
    checkQueue(arr, standardIndex, step) {
      const copy = JSON.parse(JSON.stringify(arr))
      const arr1 = []
      const arr2 = []
      for (let index = standardIndex - 1; index >= 0; index--) {
        copy[index][this.to] = copy[index + 1][this.from] - step
        if (copy[index][this.from] < copy[index][this.to]) {
          // 满足条件
          arr1.unshift(copy[index])
          continue
        } else {
          copy[index][this.from] = copy[index][this.to] - step
          arr1.unshift(copy[index])
          continue
        }
      }
      for (let index = standardIndex + 1; index <= copy.length - 1; index++) {
        copy[index][this.from] = copy[index - 1][this.to] + step
        if (copy[index][this.from] < copy[index][this.to]) {
          // 满足条件
          arr2.push(copy[index])
          continue
        } else {
          if (index === copy.length - 1 && copy[index][this.to] === 0) {
            arr2.push(copy[index])
            continue
          }
          copy[index][this.to] = copy[index][this.from] + step
          arr2.push(copy[index])
          continue
        }
      }
      const result = [...arr1, arr[standardIndex], ...arr2]
      return result
    },
    handleRangeInputFrom(index, event, arr) {
      const copy = JSON.parse(JSON.stringify(arr))
      const newVal = Number(event.target.value)
      const prevVal = this.interval[index][this.from] // 获取变化前的值
      if (newVal < 0 || newVal % this.step !== 0 || (newVal === 0 && index !== 0)) {
        copy[index][this.from] = prevVal
        this.$emit('update:configArr', copy)
        return
      }
      copy[index][this.from] = newVal
      if (copy[index][this.from] >= copy[index][this.to] && !(copy[index][this.to] === 0 && index === copy.length - 1)) {
        copy[index][this.to] = copy[index][this.from] + this.step
      }
      const res = this.checkQueue(copy, index, this.step)
      this.$emit('update:configArr', res)
    },
    handleUp() {
      this.$forceUpdate()
      this.intervalArr.sort((a, b) => a.rangeValue - b.rangeValue)
    },
    deleteItem(item, index) {
      const copy = JSON.parse(JSON.stringify(this.interval))
      copy.splice(index, 1)
      if (index !== 0) {
        copy[index - 1][this.to] = item[this.to]
      }
      this.$emit('update:configArr', copy)
    }
  }
}
</script>

  <!-- Add "scoped" attribute to limit CSS to this component only -->
  <style scoped>

  .inputNumber{
    -webkit-appearance: none;
    background-color: #fff;
    background-image: none;
    border-radius: 4px;
    border: 1px solid #dcdfe6;
    box-sizing: border-box;
    color: #606266;
    display: inline-block;
    font-size: inherit;
    height: 40px;
    line-height: 40px;
    outline: none;
    padding: 0 15px;
    transition: border-color .2s cubic-bezier(.645,.045,.355,1);
    width: 100px;
    margin-left: 5px;
  }
  .action-button{
    display: inline-block;
    line-height: 1;
    white-space: nowrap;
    cursor: pointer;
    background: #fff;
    border: 1px solid #dcdfe6;
    color: #606266;
    -webkit-appearance: none;
    text-align: center;
    box-sizing: border-box;
    outline: none;
    margin: 0;
    transition: .1s;
    font-weight: 500;
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    padding: 12px 20px;
    font-size: 14px;
    border-radius: 4px;
    color: #fff;
    background-color: #409eff;
    border-color: #409eff;
  }
  .action-button:hover{
    opacity: 0.8;
  }
  .action-button:disabled{
    cursor: not-allowed;
    color: #fff;
    /* background-color: #fab6b6; */
    /* border-color: #fab6b6; */
    opacity: .5;
  }
  .action-button + .action-button{
    margin-left: 5px;
  }
  .warning{
    background-color: #f56c6c;
    border-color: #f56c6c;
  }
  .settings{
    margin-right: 10px;
  }
  .line-wrap{
    /* width: 800px; */
    min-height: 80px;
    border-radius: 5px;
    background-color: #f0f2f5;
    margin-top: 10px;
    padding: 30px 15px;
    position: relative;
  }

  .line{
    width: 100%;
    display: flex;
    justify-content: center;
    position: relative;
    /* align-items: center; */
  }
  .min{
    position: absolute;
    left: 10px;
    top: 10px;
  }
  .max{
    position: absolute;
    right: 10px;
    top: 10px;
  }

  [type="range"] {
    /* -webkit-appearance: none; */
      /* appearance: none; */
      margin: 0;
      outline: 0;
      background-color: transparent;
      pointer-events: none;
      outline: none;
      height: 8px;
  }
  [type="range"]::-webkit-slider-thumb {
      pointer-events: auto;
  }
  [type="range"]::-moz-range-thumb {
      pointer-events: auto;
  }
  [type="range"]::-webkit-slider-container{
    border-radius: 15px;
    height: 8px;
  }
  input[type="range"]::-webkit-slider-runnable-track{
    background: none;
  }
  input[type="range"]::-moz-range-track{
    /* background: none; */
  }
  input[type="range"]::-ms-track{
    border-color: transparent;
  }

  .range{
    width: 90%;
    height: 8px;
  }
  .range::-webkit-slider-container{
    background-color: #fff;
  }
  .track-hidden{
    position: absolute;
    left: 5%;
    top: 0;
    width: 90%;
  }
  .track-hidden{
    -webkit-appearance: none;
    appearance: none;
  }
  .range-input-wrapper{
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    /* justify-content: space-around; */
  }
  .range-input-block{
    /* width: 33.33%; */
    /* padding: 10px ; */
    box-sizing: border-box;
    /* margin-right: 20px; */
    /* background-color: #fff; */
    /* display: flex; */
    /* align-items: center; */
    /* justify-content: center; */
    margin-bottom: 8px;
    width: 100%;
  }
  .flex{
    display: flex;
    align-items: center;
    /* justify-content: space-around; */
  }
  .range-outer{
    /* width: 90%; */
    height: 40px;
    background-color: #fff;
    background-image: none;
    border-radius: 4px;
    border: 1px solid #dcdfe6;
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: space-around;
    position:relative;
  }
  .range-input{
    width: 38%;
    text-align: center;
    -webkit-appearance: none;
    background-color: #fff;
    background-image: none;
    /* border-radius: 4px; */
    border: none;
    box-sizing: border-box;
    color: #606266;
    font-size: 16px;
    height: 25px;
    line-height: 25px;
    outline: none;
    padding: 0 15px;
    transition: border-color .2s cubic-bezier(.645,.045,.355,1);
  }
  .range-input::placeholder{
    line-height: 25px;
    font-size: 14px;
  }
  .sym{
    height: 25px;
    line-height: 25px;
    text-align: center;
    font-size: 14px;
    color: #303133;
  }
  .range-outer:hover .delete-action{
    display:block;
  }
  .delete-action{
    width: 15px;
    height: 15px;
    position: absolute;
    right: 1px;
    top: 2px;
    display:none;
  }
  .form{
    display: flex;
    align-items: center;
    justify-content: space-around;
  }
  /* Chrome, Safari, Edge, Opera */
  .no-arrow::-webkit-outer-spin-button,
  .no-arrow::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
  /* Firefox */
  .no-arrow[type=number] {
    -moz-appearance: textfield;
  }
  .toast{
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 80px;
    height: 50px;
    border-radius: 20px;
    background-color: rgba(000, 000, 000, 0.78);
    z-index: 10;
    color: #fff;
    text-align: center;
    line-height: 50px;
  }

  </style>

